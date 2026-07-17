---
title: "Entraînez-vous avec SQL et créez votre BDD"
subtitle: "Conception et analyse d’une base SQLite de transactions immobilières"
project_number: 3
order: 3
status: "Terminé"

summary: >-
  Conception d’une base de données relationnelle consacrée aux transactions
  immobilières françaises du premier semestre 2020. Le projet comprend la
  préparation de plusieurs sources publiques, la modélisation de sept tables,
  l’implémentation dans SQLite, la correction d’anomalies d’intégrité
  référentielle et la réalisation de douze analyses métier en SQL.

domain:
  - Data Engineering
  - Base de données
  - SQL
  - Modélisation relationnelle
  - Qualité des données
  - Analyse immobilière

technologies:
  - SQL
  - SQLite
  - SQLiteStudio
  - SQL Power Architect
  - Excel
  - Python
  - PowerShell
  - Git
  - GitHub

github_url: "https://github.com/ericginez/real-estate-sql-database"
---

## Contexte

Le projet porte sur la conception d’une base de données permettant d’analyser
les transactions immobilières réalisées en France au cours du premier semestre
2020.

Les données initiales proviennent de plusieurs sources publiques :

- les valeurs foncières ;
- le référentiel géographique français ;
- les populations communales.

Ces sources possèdent des structures et des niveaux de granularité différents.
Elles doivent être préparées, rapprochées et intégrées dans un modèle
relationnel cohérent avant de pouvoir être interrogées avec SQL.

## Objectifs

Le projet devait permettre de :

- analyser les différentes sources disponibles ;
- sélectionner les attributs nécessaires aux besoins métier ;
- concevoir un modèle relationnel normalisé ;
- construire une base SQLite ;
- définir les clés primaires et étrangères ;
- vérifier l’intégrité référentielle ;
- documenter le modèle et les règles de gestion ;
- répondre à douze besoins analytiques en SQL ;
- produire des résultats reproductibles et vérifiables.

## Périmètre des données

La base couvre les transactions comprises entre :

```text
2 janvier 2020 et 30 juin 2020
```

Elle contient sept tables :

| Table | Nombre de lignes |
|---|---:|
| `region` | 19 |
| `departement` | 109 |
| `commune` | 38 916 |
| `type_local` | 2 |
| `type_voie` | 79 |
| `bien` | 34 169 |
| `vente` | 34 169 |

La répartition des biens vendus est la suivante :

| Type de bien | Nombre |
|---|---:|
| Appartement | 31 378 |
| Maison | 2 791 |

## Préparation des données

Les sources ont d’abord été préparées dans Excel afin de construire des tables
compatibles avec le modèle relationnel.

Les principales transformations ont été :

- filtrage des transactions sur le premier semestre 2020 ;
- sélection des colonnes utiles ;
- normalisation des noms de champs ;
- création d’identifiants techniques pour les ventes et les biens ;
- rapprochement des biens avec le référentiel des communes ;
- rattachement des communes aux départements et aux régions ;
- intégration des populations communales ;
- extraction des référentiels de types de voies et de types de locaux ;
- préparation des tables destinées à l’import dans SQLite.

## Modèle relationnel

Le modèle est organisé autour de la table `vente`, reliée à la table `bien`.

La localisation géographique est représentée selon une hiérarchie composée de
trois niveaux :

```text
region
   └── departement
          └── commune
                 └── bien
                        └── vente
```

Deux référentiels complémentaires décrivent les biens :

```text
type_voie
type_local
```

Les relations du modèle cible sont :

```text
departement.code_region
    → region.code_region

commune.code_departement
    → departement.code_departement

bien.id_commune
    → commune.id_commune

bien.type_voie
    → type_voie.type_voie

bien.type_local
    → type_local.type_local

vente.id_bien
    → bien.id_bien
```

## Tables principales

### Table `vente`

La table contient les transactions immobilières :

```text
vente
├── id_vente
├── id_bien
├── date_vente
└── valeur_vente
```

### Table `bien`

La table décrit les caractéristiques et la localisation des biens :

```text
bien
├── id_bien
├── id_commune
├── no_voie
├── suffixe_no_voie
├── type_voie
├── nom_voie
├── code_postal
├── lot
├── nb_pieces
├── surface_carrez
├── surface_local
└── type_local
```

### Référentiel géographique

Le référentiel géographique est normalisé dans trois tables :

```text
region
departement
commune
```

Cette organisation évite de répéter les noms de régions et de départements dans
chaque transaction.

## Anomalie de clé étrangère

L’audit de la base SQLite initiale a révélé une erreur dans la définition de la
clé étrangère de la table `vente`.

La contrainte initiale était une auto-référence :

```text
vente.id_bien → vente.id_vente
```

La relation attendue était :

```text
vente.id_bien → bien.id_bien
```

L’anomalie était difficile à détecter, car les 34 169 transactions vérifiaient
toutes :

```text
id_vente = id_bien
```

La contrainte incorrecte trouvait donc systématiquement une correspondance dans
la table `vente`.

La table a été reconstruite afin de rétablir la relation correcte vers
`bien.id_bien`.

## Normalisation des types de voies

La colonne `bien.type_voie` contenait 940 chaînes vides.

Ces valeurs ne correspondaient pas à un code de voie, mais à une information
inconnue.

Elles ont été remplacées par des valeurs SQL `NULL` :

```sql
UPDATE bien
SET type_voie = NULL
WHERE TRIM(COALESCE(type_voie, '')) = '';
```

Après correction :

| Contrôle | Résultat |
|---|---:|
| Chaînes vides | 0 |
| Valeurs `NULL` | 940 |
| Codes non vides absents du référentiel | 0 |

## Valeurs foncières égales à zéro

La base contient 18 transactions dont la valeur foncière est égale à zéro.

Ces lignes ont été conservées afin de préserver la traçabilité des données
sources.

Elles sont néanmoins exclues des calculs monétaires :

```sql
WHERE valeur_vente > 0
```

Elles restent incluses dans les analyses portant uniquement sur le nombre de
transactions.

## Populations communales manquantes

La population n’est pas renseignée pour 3 925 communes du référentiel.

Aucune de ces communes n’est associée à un bien ou à une transaction dans le
périmètre analysé.

Ces valeurs manquantes n’affectent donc pas les résultats immobiliers.

La requête calculant le nombre de transactions pour 1 000 habitants sélectionne
uniquement les communes dont la population est connue et supérieure à
10 000 habitants.

## Règles de calcul

Deux surfaces sont disponibles dans la table `bien` :

- `surface_carrez` ;
- `surface_local`.

Les règles retenues pour le calcul du prix au mètre carré sont :

| Type de bien | Surface utilisée |
|---|---|
| Appartement | `surface_carrez` |
| Maison | `surface_local` |

Les calculs monétaires appliquent également les conditions suivantes :

```sql
valeur_vente > 0
surface_carrez > 0
surface_local > 0
```

## Contrôles d’intégrité

La base corrigée a été contrôlée avec les commandes SQLite suivantes :

```sql
PRAGMA integrity_check;
PRAGMA foreign_key_check;
```

Les résultats obtenus sont :

```text
Intégrité SQLite : ok
Violations de clés étrangères : 0
```

D’autres contrôles vérifient :

- les volumes des tables ;
- l’absence de ventes sans bien ;
- l’absence de biens sans commune ;
- l’intégrité des relations géographiques ;
- l’intégrité des types de locaux ;
- l’intégrité des types de voies renseignés ;
- l’absence de surfaces nulles ou négatives ;
- le périmètre temporel des transactions ;
- la répartition des types de biens.

## Indexation

Des index ont été ajoutés sur les colonnes utilisées dans les relations et les
filtres les plus fréquents :

```text
departement.code_region
commune.code_departement
bien.id_commune
bien.type_voie
bien.type_local
vente.id_bien
vente.date_vente
```

Cette indexation facilite les jointures entre les tables et les analyses
temporelles.

## Requêtes analytiques

Douze requêtes SQL répondent aux besoins métier.

### 1. Nombre total d’appartements vendus

La première requête calcule le nombre d’appartements vendus au premier semestre
2020.

Résultat :

```text
31 378 appartements
```

### 2. Ventes d’appartements par région

La requête agrège les ventes par région.

L’Île-de-France arrive en première position avec :

```text
13 995 ventes
```

### 3. Répartition selon le nombre de pièces

La requête calcule le nombre et la proportion d’appartements vendus pour chaque
nombre de pièces.

Les appartements de deux pièces représentent la catégorie la plus fréquente :

```text
9 783 ventes
31,18 %
```

### 4. Prix moyen au mètre carré par département

Les dix départements présentant les prix moyens les plus élevés sont classés
par ordre décroissant.

Paris arrive en première position avec :

```text
12 056,80 €/m²
```

### 5. Prix moyen des maisons en Île-de-France

Le prix moyen au mètre carré des maisons vendues en Île-de-France est :

```text
3 997,71 €/m²
```

### 6. Appartements les plus chers

La requête retourne les dix appartements ayant les valeurs foncières les plus
élevées, accompagnés de leur surface, de leur commune, de leur département et
de leur région.

### 7. Évolution trimestrielle des ventes

Le nombre de transactions est comparé entre les deux premiers trimestres de
2020 :

| Période | Nombre de ventes |
|---|---:|
| Premier trimestre | 16 776 |
| Deuxième trimestre | 17 393 |

Le taux d’évolution est :

```text
+3,68 %
```

### 8. Classement régional des grands appartements

Les régions sont classées selon le prix moyen au mètre carré des appartements
de plus de quatre pièces.

L’Île-de-France arrive en première position avec :

```text
8 770,44 €/m²
```

### 9. Communes ayant au moins cinquante ventes

La requête identifie les communes ayant enregistré au moins cinquante ventes au
premier trimestre 2020.

Elle retourne 48 communes.

### 10. Écart entre appartements de deux et trois pièces

Les prix moyens obtenus sont :

| Catégorie | Prix moyen |
|---|---:|
| Deux pièces | 4 908,57 €/m² |
| Trois pièces | 4 299,88 €/m² |

L’écart des trois-pièces par rapport aux deux-pièces est :

```text
−12,40 %
```

### 11. Top trois des communes dans cinq départements

La requête classe les trois communes ayant la valeur foncière moyenne la plus
élevée dans les départements suivants :

```text
06, 13, 33, 59 et 69
```

### 12. Transactions pour 1 000 habitants

La dernière requête classe les vingt communes ayant le plus de transactions
pour 1 000 habitants parmi celles qui dépassent 10 000 habitants.

Paris 2e arrondissement arrive en première position avec :

```text
5,84 transactions pour 1 000 habitants
```

## Résultats principaux

| Indicateur | Résultat |
|---|---:|
| Appartements vendus | 31 378 |
| Ventes au premier trimestre | 16 776 |
| Ventes au deuxième trimestre | 17 393 |
| Évolution T2 par rapport au T1 | +3,68 % |
| Ventes d’appartements en Île-de-France | 13 995 |
| Prix moyen des maisons en Île-de-France | 3 997,71 €/m² |
| Prix moyen le plus élevé par département | Paris : 12 056,80 €/m² |
| Écart entre appartements de trois et deux pièces | −12,40 % |

## Organisation du dépôt

```text
.
├── docs/
│   ├── conception.md
│   ├── dictionnaire-donnees.md
│   └── resultats-analytiques.md
├── model/
│   └── modele-relationnel.architect
├── sql/
│   ├── controles-integrite.sql
│   ├── requetes-analytiques.sql
│   └── schema.sql
├── .gitattributes
├── .gitignore
└── README.md
```

## Données non publiées

Les sources, les classeurs transformés et la base SQLite sont conservés
localement dans le dossier :

```text
local_data/
```

Ce dossier est exclu du dépôt Git.

Le dépôt public contient uniquement :

- le modèle relationnel ;
- le schéma SQL ;
- les contrôles d’intégrité ;
- les requêtes analytiques ;
- les résultats agrégés ;
- la documentation technique.

## Livrables

Le projet comprend :

- un modèle relationnel SQL Power Architect ;
- un schéma SQLite reproductible ;
- une base SQLite locale ;
- des contrôles d’intégrité SQL ;
- douze requêtes analytiques ;
- un dictionnaire de données ;
- une documentation de conception ;
- un document présentant les résultats complets.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- préparation de données issues de plusieurs sources ;
- conception d’un modèle relationnel ;
- normalisation d’une base de données ;
- définition de clés primaires et étrangères ;
- implémentation d’une base SQLite ;
- analyse des contraintes d’intégrité ;
- diagnostic et correction d’une clé étrangère ;
- normalisation des valeurs manquantes ;
- développement de requêtes SQL analytiques ;
- utilisation des jointures ;
- création de requêtes d’agrégation ;
- utilisation de CTE ;
- utilisation de fonctions de fenêtrage ;
- calcul d’indicateurs métier ;
- optimisation avec des index ;
- documentation d’un modèle de données ;
- gestion d’un dépôt GitHub.

## Limites

Le projet présente plusieurs limites :

- les données couvrent uniquement le premier semestre 2020 ;
- la préparation initiale a été effectuée dans Excel ;
- la base SQLite n’est pas publiée dans le dépôt ;
- les valeurs foncières égales à zéro ne peuvent pas être interprétées comme des
  prix de marché ;
- certaines mutations peuvent concerner plusieurs lots ou plusieurs biens ;
- le modèle actuel associe une vente à un seul bien ;
- aucune historisation pluriannuelle n’est disponible ;
- les contrôles ne sont pas encore exécutés dans une intégration continue.

## Évolutions possibles

Les évolutions envisagées comprennent :

- automatiser l’import des données avec Python ;
- intégrer plusieurs années de valeurs foncières ;
- gérer les mutations portant sur plusieurs biens ;
- ajouter une table calendrier ;
- produire des vues analytiques ;
- migrer la base vers PostgreSQL ;
- automatiser les contrôles dans une intégration continue ;
- ajouter des tests de non-régression SQL ;
- exposer les résultats dans un outil de visualisation ;
- mettre en place des contrôles statistiques sur les prix et les surfaces.