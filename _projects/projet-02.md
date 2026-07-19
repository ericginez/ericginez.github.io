---
title: "Analysez les données de systèmes éducatifs"
subtitle: "Exploration des données EdStats et présélection de pays"
project_number: 2
order: 2
status: "Terminé"

summary: >-
  Analyse exploratoire des données éducatives EdStats afin d’identifier des
  pays présentant un potentiel pour le développement international d’une offre
  de formation en ligne. Le projet comprend l’exploration de cinq sources,
  le nettoyage des données, la sélection d’indicateurs selon des critères
  métier et statistiques, l’analyse des corrélations et la production d’un
  classement exploratoire des pays.

domain:
  - Data Engineering
  - Analyse exploratoire
  - Qualité des données
  - Éducation
  - Statistiques
  - Aide à la décision

technologies:
  - Python
  - pandas
  - NumPy
  - Matplotlib
  - Seaborn
  - Jupyter Notebook
  - Poetry
  - PowerShell
  - Git
  - GitHub

github_url: "https://github.com/ericginez/education-systems-analysis"
---

## Contexte

Une entreprise spécialisée dans la formation en ligne souhaite étudier les
possibilités de développement de son activité à l’international.

Avant d’engager une étude de marché complète, elle cherche à identifier des
pays présentant un environnement éducatif, économique, démographique et
numérique favorable.

Le projet consiste à exploiter les données EdStats afin de produire une
première présélection de pays.

Cette analyse constitue un outil exploratoire d’aide à la décision. Elle ne
remplace pas une étude commerciale complète.

## Présentation

- [Consulter la présentation de soutenance au format PDF](https://github.com/ericginez/education-systems-analysis/blob/main/presentation/projet-02-analyse-systemes-educatifs.pdf)

## Objectifs

Le projet devait permettre de :

- explorer plusieurs sources de données éducatives ;
- analyser leur structure et leur niveau de qualité ;
- identifier les valeurs manquantes et les colonnes inutiles ;
- exclure les agrégats qui ne représentent pas des pays ;
- sélectionner des indicateurs pertinents selon des critères métier ;
- réduire le volume des données analysées ;
- identifier les indicateurs statistiquement redondants ;
- étudier la distribution des indicateurs retenus ;
- produire un classement exploratoire des pays ;
- documenter la méthodologie et les limites de l’analyse.

## Sources analysées

Le jeu de données EdStats comprend cinq fichiers :

| Fichier | Contenu |
|---|---|
| `EdStatsCountry.csv` | Référentiel des pays et territoires |
| `EdStatsCountry-Series.csv` | Métadonnées par pays et indicateur |
| `EdStatsData.csv` | Valeurs historiques et projections |
| `EdStatsFootNote.csv` | Notes associées aux observations |
| `EdStatsSeries.csv` | Dictionnaire des indicateurs |

Le fichier principal contient initialement :

| Mesure | Valeur |
|---|---:|
| Observations | 886 930 |
| Colonnes | 70 |
| Pays et agrégats | 241 |
| Indicateurs disponibles | 3 665 |
| Période disponible | 1970 à 2100 |

## Exploration des données

Chaque source a été étudiée afin d’identifier :

- son nombre de lignes et de colonnes ;
- la nature de ses variables ;
- ses identifiants ;
- ses valeurs manquantes ;
- ses colonnes entièrement vides ;
- ses doublons ;
- son niveau de granularité ;
- ses relations avec les autres fichiers.

Cette première analyse a permis de distinguer les données directement utiles
des métadonnées et des informations complémentaires.

## Nettoyage initial

Les colonnes entièrement vides ont été supprimées.

Les agrégats géographiques, régionaux ou économiques qui ne correspondent pas
à des pays ont également été retirés du périmètre.

Le fichier principal passe ainsi de :

| Étape | Nombre de lignes |
|---|---:|
| Données initiales | 886 930 |
| Après exclusion des agrégats | 795 305 |

Les fichiers nettoyés intermédiaires ont été conservés localement afin de
préserver la traçabilité des transformations.

## Sélection métier des indicateurs

Les indicateurs ont été sélectionnés selon leur pertinence pour une offre de
formation en ligne.

Les dimensions étudiées portent principalement sur :

- l’accès à Internet ;
- l’utilisation des services numériques ;
- le niveau économique des populations ;
- le niveau d’instruction ;
- la scolarisation secondaire ;
- l’enseignement supérieur ;
- la taille actuelle ou projetée des classes d’âge visées.

Cette sélection a permis de réduire fortement le nombre d’indicateurs avant
l’analyse statistique.

## Réduction temporelle

Les années retenues sont espacées de cinq ans :

```text
1970, 1975, 1980, 1985, 1990, 1995, 2000,
2005, 2010, 2015, 2020, 2025, 2030 et 2035
```

Cette période permet d’étudier les tendances historiques tout en intégrant les
projections disponibles à moyen terme.

## Analyse des corrélations

Une matrice de corrélation de Pearson a été calculée sur seize indicateurs.

Cette analyse permet d’identifier les variables apportant une information
similaire et de limiter les redondances dans le jeu de données final.

Après l’application des critères métier et statistiques, neuf indicateurs ont
été conservés :

```text
IT.NET.USER.P2
NY.GNP.PCAP.CD
BAR.SEC.ICMP.1519.ZS
BAR.SEC.ICMP.2024.ZS
BAR.SEC.SCHL.2024
BAR.TER.ICMP.1519.ZS
BAR.TER.ICMP.2024.ZS
PRJ.POP.1519.3.MF
PRJ.POP.1519.4.MF
```

## Analyse des distributions

Les neuf indicateurs restants ont fait l’objet d’une analyse descriptive.

Pour chaque indicateur, le notebook calcule notamment :

- le nombre de valeurs disponibles ;
- la moyenne ;
- l’écart-type ;
- les quartiles ;
- la valeur minimale ;
- la valeur maximale.

Des histogrammes permettent également d’observer la forme des distributions et
les différences d’échelle entre les indicateurs.

## Construction du classement

Pour chaque couple pays-indicateur, la valeur maximale disponible parmi les
années retenues est calculée.

La méthode applique ensuite les étapes suivantes :

1. classement décroissant des pays pour chaque indicateur ;
2. sélection des cinq premières positions ;
3. regroupement des neuf classements ;
4. comptage du nombre d’apparitions de chaque pays.

Le résultat synthétique contient :

| Mesure | Valeur |
|---|---:|
| Indicateurs | 9 |
| Pays retenus par indicateur | 5 |
| Lignes du classement | 45 |
| Pays distincts représentés | 33 |

## Résultats

La Slovénie apparaît dans trois classements, soit la fréquence la plus élevée.

Les pays suivants apparaissent deux fois :

- l’Australie ;
- les Bermudes ;
- le Botswana ;
- la Chine ;
- la Corée du Sud ;
- Cuba ;
- l’Inde ;
- l’Islande ;
- la Norvège ;
- la République tchèque.

Les autres pays apparaissent une seule fois.

La présence de petits territoires comme les Bermudes montre que les résultats
doivent être replacés dans un contexte commercial plus large avant toute
décision.

## Pipeline d’analyse

```text
Sources EdStats
      │
      ▼
Exploration et diagnostic
      │
      ▼
Nettoyage des colonnes inutiles
      │
      ▼
Exclusion des agrégats géographiques
      │
      ▼
Sélection métier des indicateurs
      │
      ▼
Réduction de la période d’analyse
      │
      ▼
Analyse des corrélations de Pearson
      │
      ▼
Sélection de neuf indicateurs
      │
      ▼
Analyse descriptive
      │
      ▼
Classement des pays par indicateur
```

## Fichiers produits

Le dossier `data/processed` contient six résultats publics :

| Fichier | Contenu |
|---|---|
| `indicateurs-selectionnes.csv` | Première sélection d’indicateurs |
| `correlation-pearson.csv` | Matrice de corrélation |
| `donnees-analyse.csv` | Données correspondant aux neuf indicateurs finaux |
| `classement-pays.csv` | Données enrichies avec la valeur maximale |
| `top-5-pays-par-indicateur.csv` | Cinq premiers pays de chaque indicateur |
| `frequence-pays.csv` | Nombre d’apparitions de chaque pays |

## Organisation du dépôt

```text
.
├── data/
│   └── processed/
│       ├── classement-pays.csv
│       ├── correlation-pearson.csv
│       ├── donnees-analyse.csv
│       ├── frequence-pays.csv
│       ├── indicateurs-selectionnes.csv
│       └── top-5-pays-par-indicateur.csv
├── docs/
│   └── methodologie.md
├── analyse-systemes-educatifs.ipynb
├── poetry.lock
├── pyproject.toml
├── .gitattributes
├── .gitignore
└── README.md
```

## Données non publiées

Les fichiers EdStats bruts et les données intermédiaires sont conservés
localement dans :

```text
local_data/
```

Les notebooks de travail correspondant aux différentes étapes sont conservés
dans :

```text
local_notebooks/
```

Ces dossiers sont exclus du dépôt Git en raison du volume des données et afin de
ne publier que les livrables utiles.

## Livrables

Le projet comprend :

- un notebook Jupyter consolidé ;
- une analyse exploratoire complète ;
- un pipeline documenté de nettoyage et de sélection ;
- une matrice de corrélation de Pearson ;
- un jeu de données réduit à neuf indicateurs ;
- un classement des cinq premiers pays par indicateur ;
- une synthèse de la fréquence d’apparition des pays ;
- une documentation méthodologique ;
- un environnement Python reproductible avec Poetry ;
- un dépôt GitHub documenté.
- une présentation de soutenance au format PDF.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- exploration de jeux de données volumineux ;
- analyse de la structure de plusieurs sources ;
- traitement des valeurs manquantes ;
- nettoyage et filtrage de données ;
- identification d’agrégats géographiques ;
- sélection de variables selon des critères métier ;
- réduction de la dimension d’un jeu de données ;
- manipulation de données avec pandas ;
- calcul de statistiques descriptives ;
- création de visualisations avec Matplotlib et Seaborn ;
- analyse des corrélations ;
- construction de classements ;
- production de fichiers de résultats ;
- documentation d’une démarche analytique ;
- gestion des dépendances avec Poetry ;
- gestion d’un dépôt GitHub.

## Limites

Le projet présente plusieurs limites :

- la valeur maximale est calculée sur l’ensemble de la période ;
- les données historiques et les projections sont analysées conjointement ;
- les indicateurs ne sont pas normalisés dans un score global ;
- tous les indicateurs ont implicitement le même poids ;
- certaines données sont manquantes ;
- les territoires de petite taille ne sont pas exclus ;
- le classement ne tient pas compte de la langue ;
- la concurrence locale n’est pas analysée ;
- la réglementation de la formation n’est pas intégrée ;
- la taille réelle du marché n’est pas évaluée ;
- les données brutes ne sont pas publiées dans le dépôt.

## Évolutions possibles

Les évolutions envisagées comprennent :

- actualiser les données EdStats ;
- distinguer explicitement les observations des projections ;
- normaliser les indicateurs ;
- construire un score multicritère pondéré ;
- intégrer la taille du marché potentiel ;
- ajouter les critères linguistiques ;
- analyser la concurrence locale ;
- intégrer les contraintes réglementaires ;
- exclure ou traiter séparément les micro-États et territoires ;
- automatiser le pipeline de préparation ;
- ajouter des tests de qualité des données ;
- produire un tableau de bord interactif ;
- comparer les résultats avec d’autres sources internationales.
