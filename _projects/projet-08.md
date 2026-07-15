---
title: "Construisez et testez une infrastructure de données"
subtitle: "Pipeline météorologique multi-source avec dbt, PostgreSQL, Docker et AWS"
project_number: 8
order: 8
status: "Terminé"

summary: >-
  Conception d’une infrastructure de données météorologiques permettant
  d’intégrer, harmoniser, tester et modéliser des observations provenant
  d’Infoclimat et de Weather Underground, puis de déployer les traitements
  dbt dans AWS.

domain:
  - Data Engineering
  - Infrastructure cloud
  - Transformation de données
  - Modélisation dimensionnelle
  - Qualité des données

technologies:
  - Python
  - PostgreSQL
  - dbt
  - Docker
  - Airbyte
  - AWS RDS
  - AWS ECR
  - AWS ECS
  - AWS EC2
  - Git
  - GitHub

github_url: "https://github.com/ericginez/weather-data-infrastructure"
---

## Contexte

Le projet consiste à construire une infrastructure de données capable
d’intégrer des observations météorologiques provenant de plusieurs
systèmes.

Deux sources sont utilisées :

- **Infoclimat**, pour les métadonnées et les observations de plusieurs
  stations météorologiques ;
- **Weather Underground**, pour les observations issues de stations
  situées à La Madeleine et à Ichtegem.

Les sources possèdent des structures, des identifiants, des formats de
dates et des unités différents. Elles doivent donc être normalisées avant
de pouvoir être exploitées dans un modèle analytique commun.

L’objectif est de mettre en place une chaîne reproductible allant de
l’ingestion des données brutes jusqu’à la création de tables de faits et
de dimensions testées avec dbt.

## Objectifs

Le projet répond aux objectifs suivants :

- collecter des données météorologiques multi-sources ;
- préparer les fichiers bruts avec Python ;
- charger les données dans PostgreSQL ;
- structurer les transformations dbt en plusieurs couches ;
- harmoniser les noms de colonnes et les unités ;
- fusionner les stations et les observations ;
- construire un modèle dimensionnel ;
- contrôler automatiquement la qualité des données ;
- conteneuriser les traitements ;
- déployer l’infrastructure dans AWS ;
- planifier l’exécution du pipeline.

## Sources de données

### Infoclimat

Les scripts Python permettent d’extraire :

- les identifiants des stations ;
- leurs noms et coordonnées géographiques ;
- leur altitude et leur type ;
- les observations météorologiques associées.

Les quatre identifiants de stations attendus dans cette source sont :

```text
00052
000R5
07015
STATIC0010
```

### Weather Underground

Les fichiers Excel Weather Underground sont convertis en fichiers
structurés avant leur chargement.

Les deux stations utilisées sont :

```text
ILAMAD25
IICHTE19
```

Les observations couvrent la période du 1er au 7 octobre 2024.

## Préparation des données

Trois scripts Python sont utilisés pour préparer les sources :

```text
convert_weatherug.py
extract_infoclimat_observations.py
extract_infoclimat_stations.py
```

Ils assurent notamment :

- la lecture des fichiers Excel et JSON ;
- la sélection des colonnes utiles ;
- la normalisation des noms ;
- la conversion des dates ;
- la préparation des fichiers chargés dans PostgreSQL.

Les fichiers de données brutes ne sont pas publiés dans le dépôt GitHub.

## Architecture dbt

Le projet dbt est organisé en trois couches :

```text
Sources brutes
     │
     ▼
Staging
     │
     ▼
Intermediate
     │
     ▼
Marts
```

Cette organisation sépare les opérations de normalisation, d’unification
et de modélisation analytique.

## Couche staging

La couche `staging` contient quatre modèles matérialisés sous forme de
vues :

```text
stg_infoclimat_stations
stg_infoclimat_observations
stg_weatherug_stations
stg_weatherug_observations
```

Cette couche effectue principalement :

- le renommage des colonnes ;
- le typage des valeurs ;
- la conversion des dates ;
- l’harmonisation des unités ;
- la sélection des attributs utiles ;
- les premiers contrôles de qualité.

Les mesures météorologiques sont standardisées dans des unités communes,
notamment :

- température en degrés Celsius ;
- humidité en pourcentage ;
- pression en hectopascals ;
- vitesse du vent en kilomètres par heure ;
- précipitations en millimètres.

## Couche intermédiaire

La couche `intermediate` contient deux modèles :

```text
int_weather_stations
int_weather_observations
```

Elle rassemble les données normalisées des deux systèmes.

Les modèles intermédiaires permettent notamment :

- d’unifier les schémas Infoclimat et Weather Underground ;
- d’ajouter la source d’origine ;
- d’aligner les attributs des stations ;
- de consolider les observations ;
- de préparer les tables analytiques finales.

## Modèle dimensionnel

La couche `marts` produit deux tables.

### Dimension des stations

```text
dim_weather_stations
```

Elle contient les informations descriptives des stations :

- identifiant ;
- nom ;
- source ;
- type ;
- latitude ;
- longitude ;
- altitude ;
- licence ou métadonnée équivalente.

La dimension finale regroupe les stations provenant des deux sources.

### Table de faits des observations

```text
fact_weather_observations
```

Sa granularité est :

```text
une station à un instant d’observation
```

Elle contient notamment :

- la date et l’heure d’observation ;
- la température ;
- l’humidité ;
- la pression atmosphérique ;
- le point de rosée ;
- la vitesse moyenne du vent ;
- la direction du vent ;
- les mesures de précipitations ;
- le système source.

## Qualité des données

Les contrôles sont déclarés dans les fichiers `schema.yml` des couches
staging et marts.

Ils couvrent plusieurs familles de tests.

### Complétude

Les tests `not_null` contrôlent la présence des informations critiques :

- identifiants des stations ;
- dates d’observation ;
- températures ;
- taux d’humidité ;
- pressions ;
- vitesses du vent ;
- noms et coordonnées des stations.

### Unicité

Les tests vérifient :

- l’unicité de chaque identifiant de station ;
- l’unicité du couple `station_id` et `observed_at`.

Cette dernière règle garantit qu’une même station ne possède pas deux
observations pour un même instant.

### Valeurs autorisées

Les systèmes sources autorisés sont :

```text
infoclimat
weatherug
```

Les identifiants attendus des stations sont également contrôlés.

### Plages métier

Les mesures doivent respecter des intervalles cohérents :

| Mesure | Minimum | Maximum |
|---|---:|---:|
| Température | -50 °C | 60 °C |
| Humidité | 0 % | 100 % |
| Pression | 850 hPa | 1 100 hPa |
| Vitesse moyenne du vent | 0 km/h | 150 km/h |

Ces contrôles utilisent les tests génériques du package `dbt-utils`.

### Intégrité référentielle

Un test `relationships` vérifie que chaque observation présente dans la
table de faits correspond à une station de la dimension.

## Environnement local

Une instance PostgreSQL 15 peut être lancée localement avec Docker
Compose.

```text
Conteneur : projet8_postgres
Base       : weather_db
Port       : 5432
```

Cet environnement permet de développer et tester le projet avant son
déploiement dans le cloud.

Les principales commandes dbt sont :

```powershell
poetry run dbt deps
poetry run dbt debug
poetry run dbt run
poetry run dbt test
poetry run dbt build
```

## Conteneurisation

Le projet dbt est empaqueté dans une image Docker.

L’image contient :

- Python ;
- dbt-postgres ;
- les dépendances du projet ;
- les modèles SQL ;
- les macros ;
- les tests de qualité.

Cette conteneurisation rend le traitement reproductible entre
l’environnement local et AWS.

## Architecture cloud AWS

L’architecture cloud repose sur plusieurs services AWS.

### Amazon EC2

Une instance EC2 héberge Airbyte, utilisé pour l’ingestion et le transfert
des données vers PostgreSQL.

### Amazon RDS for PostgreSQL

Amazon RDS fournit la base de données PostgreSQL managée utilisée par le
pipeline.

Elle centralise :

- les tables brutes ;
- les vues de staging ;
- les modèles intermédiaires ;
- les tables analytiques finales.

### Amazon ECR

L’image Docker du projet dbt est publiée dans Amazon Elastic Container
Registry.

ECR permet de versionner et distribuer l’image utilisée pour exécuter les
transformations.

### Amazon ECS

Une tâche ECS lance le conteneur dbt et exécute les transformations sur
la base PostgreSQL RDS.

La tâche est planifiée afin d’automatiser l’actualisation des modèles.

L’infrastructure a été déployée dans la région :

```text
eu-west-3
```

## Sécurité

Les informations sensibles ne sont pas versionnées dans GitHub.

Le `.gitignore` exclut notamment :

- les clés privées EC2 ;
- les fichiers `.pem` ;
- les profils dbt contenant les identifiants ;
- les variables d’environnement ;
- les fichiers de données brutes ;
- les logs ;
- les packages téléchargés ;
- les artefacts générés dans `target`.

La clé privée utilisée pour accéder à EC2 reste uniquement sur la machine
locale.

## Résultats

Le projet aboutit à une infrastructure comprenant :

- deux sources météorologiques ;
- quatre modèles de staging ;
- deux modèles intermédiaires ;
- une dimension des stations ;
- une table de faits des observations ;
- six stations attendues dans le modèle consolidé ;
- des tests automatisés de complétude ;
- des tests d’unicité ;
- des tests de plages métier ;
- un contrôle d’intégrité référentielle ;
- une documentation générée par dbt ;
- une image Docker publiée dans Amazon ECR ;
- une tâche dbt planifiée dans Amazon ECS.

## Difficultés rencontrées

### Harmonisation des sources

Infoclimat et Weather Underground utilisent des structures différentes.
La création d’un schéma commun a nécessité de normaliser les colonnes,
les types et les unités avant leur fusion.

### Gestion des relations dbt

Les tests de relation entre la table de faits et la dimension des
stations ont nécessité d’aligner précisément les identifiants générés
dans les différentes couches.

### Exécution dans le cloud

Le passage d’un environnement local à AWS implique de configurer :

- les accès réseau ;
- les groupes de sécurité ;
- les rôles IAM ;
- la connexion entre ECS et RDS ;
- l’accès à l’image stockée dans ECR ;
- la transmission sécurisée des paramètres de connexion.

### Gestion des fuseaux horaires

Les dates des observations et les horaires d’exécution doivent être
interprétés de manière cohérente entre les sources, PostgreSQL et
l’infrastructure AWS.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- conception d’une infrastructure de données ;
- ingestion de données multi-sources ;
- préparation de données avec Python ;
- transformation SQL avec dbt ;
- structuration en couches analytiques ;
- modélisation dimensionnelle ;
- création de tests automatisés ;
- contrôle de la qualité des données ;
- utilisation de PostgreSQL ;
- conteneurisation avec Docker ;
- déploiement sur AWS ;
- utilisation d’Amazon RDS, ECR, ECS et EC2 ;
- gestion des secrets et des accès ;
- documentation technique d’un pipeline.