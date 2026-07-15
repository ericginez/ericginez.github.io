---
title: "Concevez et analysez une base de données NoSQL"
subtitle: "Analyse d’annonces locatives avec MongoDB, Polars et Power BI"
project_number: 7
order: 7
status: "Terminé"

summary: >-
  Conception et analyse d’une base documentaire MongoDB contenant près
  de 96 000 annonces parisiennes, avec extraction ciblée via PyMongo,
  transformations analytiques avec Polars et restitution des indicateurs
  dans Power BI.

domain:
  - Data Engineering
  - Base de données NoSQL
  - Analyse de données
  - Business Intelligence
  - Modélisation documentaire

technologies:
  - MongoDB
  - PyMongo
  - Polars
  - Python
  - Jupyter Notebook
  - Power BI
  - Poetry
  - Git LFS
  - Git
  - GitHub

github_url: "https://github.com/ericginez/mongodb-rental-listings-analysis"
---

## Contexte

Le projet consiste à concevoir et analyser une base de données NoSQL
contenant des annonces de locations de courte durée pour Paris et Lyon.

Chaque annonce possède un grand nombre d’attributs relatifs :

- au logement ;
- à son emplacement ;
- à l’hôte ;
- à la capacité d’accueil ;
- aux disponibilités ;
- aux conditions de réservation ;
- aux avis laissés par les voyageurs.

La structure de ces données est semi-structurée et susceptible d’évoluer
entre les différentes extractions. Une base documentaire MongoDB est donc
adaptée à leur stockage et à leur interrogation.

L’analyse finale porte principalement sur les annonces parisiennes, avec
une extraction depuis MongoDB, des traitements réalisés avec Polars et
une restitution dans Power BI.

## Objectifs

Le projet répond aux objectifs suivants :

- concevoir une base documentaire MongoDB ;
- importer un volume important de données semi-structurées ;
- sélectionner uniquement les champs nécessaires à l’analyse ;
- interroger MongoDB avec PyMongo ;
- convertir les documents en DataFrame Polars ;
- nettoyer et typer les données extraites ;
- produire des indicateurs sur les logements et les hôtes ;
- analyser la répartition géographique des annonces ;
- étudier la disponibilité des logements ;
- comparer les annonces des super-hôtes et des autres hôtes ;
- restituer les résultats dans un rapport Power BI ;
- expérimenter plusieurs architectures MongoDB locales.

## Sources de données

Le projet utilise deux fichiers d’annonces :

```text
listings_Paris.csv
listings_Lyon.csv
```

Le fichier parisien représente environ 185 Mo et contient près de
96 000 annonces.

Les fichiers sources ne sont pas publiés dans le dépôt GitHub en raison
de leur volume. Le dépôt contient toutefois le dictionnaire de données
dans deux formats :

```text
Data Dictionary.csv
Data Dictionary.xlsx
```

Ce dictionnaire documente notamment :

- les identifiants des annonces ;
- les informations sur les hôtes ;
- les caractéristiques des logements ;
- les coordonnées géographiques ;
- les disponibilités ;
- les avis ;
- les conditions de réservation.

## Modèle documentaire

Chaque annonce est stockée dans MongoDB sous la forme d’un document.

Cette modélisation permet de conserver dans une même structure les
différentes familles d’informations :

```text
Annonce
├── identification
├── description
├── hôte
├── localisation
├── caractéristiques du logement
├── capacité d’accueil
├── prix
├── disponibilité
├── avis
└── conditions de réservation
```

Les documents peuvent contenir des champs absents, facultatifs ou
hétérogènes sans nécessiter une structure relationnelle rigide.

## Base MongoDB analysée

Le notebook se connecte à une instance MongoDB locale :

```python
client = MongoClient("mongodb://localhost:27017/")
```

La base et la collection interrogées sont :

```text
Base       : locparis
Collection : listings
```

Le chargement initial des fichiers CSV dans MongoDB a été réalisé en
amont du notebook.

L’analyse publiée dans le dépôt concerne la base parisienne.

## Extraction avec PyMongo

Le notebook utilise PyMongo pour interroger la collection MongoDB.

Une projection limite l’extraction aux champs nécessaires :

```python
projection = {
    "_id": 0,
    "host_is_superhost": 1,
    "room_type": 1,
    "neighbourhood_cleansed": 1,
    "number_of_reviews": 1,
    "availability_365": 1,
}
```

Les attributs retenus sont :

| Champ | Utilisation |
|---|---|
| `host_is_superhost` | Comparaison des catégories d’hôtes |
| `room_type` | Analyse par type de logement |
| `neighbourhood_cleansed` | Répartition par quartier |
| `number_of_reviews` | Analyse de l’activité des annonces |
| `availability_365` | Estimation des jours non disponibles |

Cette projection permet de réduire :

- le volume transféré depuis MongoDB ;
- la consommation mémoire ;
- le nombre de colonnes à nettoyer ;
- le temps de création du DataFrame.

## Chargement dans Polars

Les documents extraits sont chargés dans un DataFrame Polars :

```python
docs = list(collection.find({}, projection))
df = pl.DataFrame(docs)
```

Les colonnes sont ensuite converties dans des types adaptés :

```python
df = df.with_columns([
    pl.col("host_is_superhost").cast(pl.Utf8, strict=False),
    pl.col("room_type").cast(pl.Utf8, strict=False),
    pl.col("neighbourhood_cleansed").cast(pl.Utf8, strict=False),
    pl.col("number_of_reviews").cast(pl.Int64, strict=False),
    pl.col("availability_365").cast(pl.Int64, strict=False),
])
```

Polars est utilisé pour :

- filtrer les valeurs incohérentes ;
- regrouper les données ;
- calculer des médianes ;
- produire des agrégations ;
- trier les résultats ;
- préparer les indicateurs destinés au reporting.

## Indicateurs calculés

### Disponibilité par type de logement

Le nombre estimé de jours non disponibles par mois est calculé à partir
du champ `availability_365` :

```text
Jours non disponibles annuels = 365 - availability_365
```

puis :

```text
Jours non disponibles mensuels estimés
    = (365 - availability_365) / 12
```

Les résultats moyens sont :

| Type de logement | Jours non disponibles estimés par mois |
|---|---:|
| Private room | 20,93 |
| Entire home/apt | 19,74 |
| Shared room | 19,26 |
| Hotel room | 15,93 |

Cet indicateur est une estimation. Un logement peut être indisponible
parce qu’il est réservé, mais aussi parce que son hôte l’a volontairement
bloqué.

Il ne représente donc pas un taux de réservation réel.

### Médiane du nombre d’avis

La médiane du nombre d’avis pour l’ensemble des logements est :

```text
3 avis
```

La médiane est utilisée afin de limiter l’influence des annonces
présentant un nombre exceptionnellement élevé d’avis.

### Répartition des hôtes

La répartition des annonces selon le statut de l’hôte est la suivante :

| Statut | Nombre d’annonces |
|---|---:|
| Non super-hôte | 81 035 |
| Super-hôte | 14 776 |
| Valeur non renseignée | 74 |

### Comparaison des avis par catégorie d’hôte

La médiane du nombre d’avis diffère fortement selon la catégorie :

| Catégorie | Médiane du nombre d’avis |
|---|---:|
| Super-hôte | 24 |
| Non super-hôte | 2 |

Les annonces associées aux super-hôtes possèdent donc une médiane
d’avis nettement supérieure.

Cette observation décrit une corrélation, mais ne permet pas à elle seule
d’établir une relation de causalité.

## Répartition géographique

Les annonces sont regroupées par quartier parisien à partir du champ :

```text
neighbourhood_cleansed
```

Les quartiers contenant le plus grand nombre d’annonces sont :

| Quartier | Nombre de logements | Part des annonces |
|---|---:|---:|
| Buttes-Montmartre | 10 555 | 11,01 % |
| Popincourt | 8 430 | 8,79 % |
| Vaugirard | 7 802 | 8,14 % |
| Batignolles-Monceau | 6 857 | 7,15 % |
| Entrepôt | 6 558 | 6,84 % |

Buttes-Montmartre représente à lui seul environ 11 % des annonces
analysées.

## Quartiers les moins disponibles

Le projet calcule également le nombre moyen estimé de jours non
disponibles par mois pour chaque quartier.

Les dix premiers résultats sont :

| Quartier | Jours non disponibles estimés | Nombre de logements |
|---|---:|---:|
| Ménilmontant | 21,62 | 5 271 |
| Buttes-Chaumont | 21,21 | 5 465 |
| Buttes-Montmartre | 21,06 | 10 555 |
| Entrepôt | 20,99 | 6 558 |
| Popincourt | 20,95 | 8 430 |
| Gobelins | 20,76 | 3 304 |
| Reuilly | 20,61 | 4 003 |
| Panthéon | 20,18 | 3 010 |
| Vaugirard | 20,05 | 7 802 |
| Observatoire | 19,82 | 3 611 |

Cette analyse permet d’identifier les quartiers dans lesquels les
annonces présentent la plus faible disponibilité moyenne.

## Rapport Power BI

Les résultats sont restitués dans le fichier :

```text
Projet 07.pbix
```

Le rapport Power BI permet notamment de visualiser :

- la répartition des logements par quartier ;
- le poids relatif de chaque zone géographique ;
- les différents types de logements ;
- la comparaison entre super-hôtes et autres hôtes ;
- la distribution du nombre d’avis ;
- les niveaux estimés de disponibilité.

Power BI constitue la couche de restitution du projet, tandis que
MongoDB et Polars assurent respectivement le stockage et les traitements.

## Architectures MongoDB expérimentées

Deux architectures MongoDB ont été expérimentées localement :

```text
mongo-rs/
mongo-shard/
```

### Replica set

Le replica set permet de disposer de plusieurs instances MongoDB
répliquant les mêmes données.

Son intérêt principal est d’améliorer :

- la disponibilité ;
- la tolérance aux pannes ;
- la continuité de service ;
- la possibilité d’élire un nouveau nœud primaire.

### Sharding

Le sharding permet de distribuer les données entre plusieurs nœuds.

Cette architecture vise principalement à :

- répartir les volumes ;
- distribuer la charge ;
- améliorer la scalabilité horizontale ;
- traiter des jeux de données plus importants.

Les répertoires locaux contiennent les fichiers internes WiredTiger,
les journaux, les index et les métadonnées générés par MongoDB.

Ces fichiers ne constituent pas une configuration portable et ne sont
donc pas publiés dans GitHub.

Le dépôt ne fournit pas encore de fichier Docker Compose permettant de
reconstruire automatiquement ces architectures.

## Gestion des fichiers volumineux

Le rapport Power BI représente plus de 50 Mo.

Il est versionné avec Git LFS grâce à la règle :

```gitattributes
*.pbix filter=lfs diff=lfs merge=lfs -text
```

Git LFS stocke un pointeur dans l’historique Git et conserve le fichier
binaire dans un stockage adapté.

Les éléments suivants sont exclus du dépôt :

```text
listings_Paris.csv
listings_Lyon.csv
mongo-rs/
mongo-shard/
.ipynb_checkpoints/
```

Cette stratégie permet de publier le code, le notebook, le dictionnaire
de données et le rapport sans intégrer les volumes MongoDB ni les données
sources volumineuses.

## Difficultés rencontrées

### Gestion du volume de données

Le fichier parisien représente environ 185 Mo.

Son chargement complet et sa transformation nécessitent une attention
particulière à la mémoire et au volume transféré.

L’utilisation d’une projection MongoDB limite l’extraction aux champs
réellement nécessaires.

### Données semi-structurées

Les champs peuvent être manquants, vides ou stockés dans des formats
différents.

Le typage Polars est donc réalisé avec une conversion tolérante afin de
ne pas interrompre le traitement lorsqu’une valeur ne correspond pas au
type attendu.

### Interprétation de la disponibilité

Le champ `availability_365` ne permet pas de distinguer les jours
réservés des jours bloqués par l’hôte.

L’indicateur produit doit donc être interprété comme un nombre estimé de
jours non disponibles, et non comme un taux de réservation certain.

### Reproductibilité des architectures MongoDB

Les volumes WiredTiger permettent de retrouver l’état local des bases,
mais ils ne peuvent pas remplacer une configuration déclarative.

Une reconstruction automatique nécessiterait des scripts ou un fichier
Docker Compose dédié.

### Versionnement du rapport Power BI

Le fichier PBIX dépasse la taille recommandée pour un fichier Git
classique.

Git LFS est utilisé afin de le publier sans alourdir directement
l’historique du dépôt.

## Résultats

Le projet aboutit à :

- une base documentaire MongoDB d’annonces locatives ;
- une analyse de près de 96 000 annonces parisiennes ;
- une extraction optimisée avec une projection PyMongo ;
- un DataFrame Polars correctement typé ;
- une analyse par type de logement ;
- une comparaison des super-hôtes et des autres hôtes ;
- une analyse de la répartition des annonces par quartier ;
- une estimation de la disponibilité mensuelle ;
- un rapport Power BI ;
- une expérimentation du replica set ;
- une expérimentation du sharding ;
- un dépôt GitHub documenté et sécurisé ;
- un versionnement du rapport avec Git LFS.

## Limites

Le projet présente plusieurs limites :

- les fichiers de données sources ne sont pas publiés ;
- l’import des CSV dans MongoDB n’est pas automatisé dans le dépôt ;
- le notebook final analyse principalement Paris ;
- les données de Lyon ne sont pas exploitées dans l’analyse publiée ;
- les architectures replica set et shardée ne sont pas reproductibles
  à partir des seuls fichiers versionnés ;
- l’indicateur fondé sur `availability_365` reste une approximation ;
- le rapport Power BI dépend de données locales ;
- aucun test automatisé de qualité des données n’est encore fourni.

## Évolutions possibles

Le projet pourrait être enrichi avec :

- un script automatisé d’import CSV vers MongoDB ;
- une configuration Docker Compose ;
- une initialisation automatique du replica set ;
- une configuration reproductible du sharding ;
- une analyse comparative Paris-Lyon ;
- des pipelines d’agrégation exécutés directement dans MongoDB ;
- la création d’index adaptés aux requêtes ;
- une mesure des gains apportés par les index ;
- des tests automatisés de qualité ;
- un export Parquet intermédiaire ;
- un petit jeu de données d’exemple ;
- une actualisation automatisée du rapport Power BI ;
- une documentation détaillée du schéma documentaire.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- conception d’une base de données NoSQL ;
- modélisation documentaire ;
- manipulation de données semi-structurées ;
- interrogation de MongoDB avec PyMongo ;
- utilisation de projections MongoDB ;
- traitement analytique avec Polars ;
- nettoyage et typage de données ;
- création d’agrégations métier ;
- analyse géographique ;
- interprétation critique des indicateurs ;
- utilisation de Jupyter Notebook ;
- conception d’un rapport Power BI ;
- compréhension de la réplication MongoDB ;
- compréhension du sharding ;
- gestion de fichiers volumineux avec Git LFS ;
- documentation d’un projet Data Engineering.