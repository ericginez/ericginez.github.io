---
title: "Mettez en place un pipeline d’orchestration des flux"
subtitle: "Automatisation d’un pipeline de données e-commerce avec Kestra et DuckDB"
project_number: 10
order: 10
status: "Terminé"

summary: >-
  Conception d’un pipeline mensuel orchestré avec Kestra pour préparer,
  nettoyer, rapprocher et analyser des données ERP et e-commerce,
  contrôler leur qualité et produire automatiquement un rapport de
  chiffre d’affaires ainsi que des exports de vins premium et ordinaires.

domain:
  - Data Engineering
  - Orchestration
  - Transformation de données
  - Qualité des données
  - Analyse métier

technologies:
  - Kestra
  - DuckDB
  - SQL
  - Python
  - Pandas
  - OpenPyXL
  - XlsxWriter
  - PostgreSQL
  - Docker
  - Docker Compose
  - Git
  - GitHub

github_url: "https://github.com/ericginez/wine-orchestration-pipeline"
---

## Contexte

BottleNeck commercialise des vins prestigieux et dispose de données
réparties entre plusieurs systèmes.

Trois fichiers Excel alimentent le traitement :

- un fichier ERP contenant les références produits, les prix et les stocks ;
- un fichier de liaison entre les identifiants ERP et les identifiants web ;
- un fichier issu de la plateforme e-commerce contenant les ventes et les
  informations publiées en ligne.

Ces sources ne sont pas directement alignées. Elles doivent être nettoyées,
dédupliquées et rapprochées avant de pouvoir calculer des indicateurs fiables.

L’objectif est d’automatiser cette chaîne dans un workflow reproductible,
contrôlé et planifié mensuellement.

Les responsables produits doivent recevoir :

- un rapport synthétique du chiffre d’affaires ;
- un export des vins premium ;
- un export des vins ordinaires.

## Présentation

- [Consulter la présentation de soutenance au format PDF](https://github.com/ericginez/wine-orchestration-pipeline/blob/main/presentation/projet-10-pipeline-orchestration-flux.pdf)

## Objectifs

Le projet répond aux objectifs suivants :

- préparer les fichiers sources Excel ;
- convertir les sources dans des formats intermédiaires exploitables ;
- charger les données dans DuckDB ;
- nettoyer et normaliser les données ;
- supprimer les doublons ;
- rapprocher les références ERP et web ;
- contrôler la qualité des jointures ;
- vérifier l’absence de valeurs nulles critiques ;
- calculer le chiffre d’affaires par produit et global ;
- détecter les prix atypiques avec un z-score ;
- classer les vins en catégories premium et ordinaire ;
- générer automatiquement les fichiers de restitution ;
- interrompre le workflow lorsqu’un contrôle critique échoue ;
- planifier l’exécution mensuelle du pipeline ;
- superviser les tâches et les journaux dans Kestra.

## Sources de données

### Données ERP

Le fichier :

```text
data/input/Fichier_erp.xlsx
```

contient notamment :

- les identifiants produits ;
- les prix ;
- les quantités en stock ;
- les informations de référence utilisées dans l’ERP.

Après nettoyage et déduplication, cette source comprend :

```text
825 références uniques
```

### Fichier de liaison

Le fichier :

```text
data/input/fichier_liaison.xlsx
```

établit la correspondance entre les identifiants utilisés dans l’ERP et
ceux employés sur la plateforme e-commerce.

Il constitue la clé de rapprochement entre les deux systèmes.

### Données web

Le fichier :

```text
data/input/Fichier_web.xlsx
```

contient notamment :

- les identifiants web ;
- les références produit ;
- les quantités vendues ;
- les informations descriptives publiées en ligne.

Après nettoyage et déduplication, cette source comprend :

```text
714 références uniques
```

Les fichiers sources restent locaux et ne sont pas publiés dans le dépôt.

## Architecture du pipeline

Le workflow est orchestré par Kestra. Les traitements utilisent Python,
SQL et DuckDB.

```text
Fichiers Excel
├── ERP
├── Liaison
└── Web
       │
       ▼
Préparation et conversion des sources
       │
       ▼
Chargement dans DuckDB
       │
       ▼
Nettoyage des données web
       │
       ▼
Déduplication
       │
       ▼
Contrôle de l’absence de doublons
       │
       ▼
Rapprochement ERP / liaison / web
       │
       ├── contrôle des valeurs nulles
       └── contrôle des jointures
       │
       ▼
Calcul du chiffre d’affaires
       │
       ▼
Contrôle de cohérence du chiffre d’affaires
       │
       ▼
Calcul du z-score et classification
       │
       ▼
Contrôle du z-score
       │
       ▼
Exports Excel et CSV
```

Cette organisation sépare les transformations, les contrôles et les exports.
Elle permet d’identifier précisément l’étape responsable d’une anomalie.

## Workflow Kestra

Le workflow est défini dans :

```text
flows/wine_pipeline.yaml
```

Ses identifiants sont :

```yaml
id: wine_pipeline
namespace: projet10
```

Il apparaît donc dans Kestra sous le nom :

```text
projet10.wine_pipeline
```

## Planification mensuelle

Le workflow utilise un déclencheur cron :

```yaml
cron: "0 9 15 * *"
timezone: "Europe/Paris"
```

Le pipeline est exécuté automatiquement :

```text
le 15 de chaque mois à 09:00
```

Il peut également être lancé manuellement depuis l’interface Kestra.

## Étapes du traitement

L’ordre exact des tâches est :

```text
01. prepare_sources
02. sql_load_sources
03. sql_web_clean_before_dedup
04. sql_suppress_duplicates
05. test_duplicates_absence
06. sql_systems_fusion
07. test_nulls_absence
08. test_joins_consistency
09. sql_sales_revenue
10. test_sales_revenue_consistency
11. zscore
12. test_zscore
13. export_outputs
```

### Préparation des sources

Le script :

```text
scripts/00_prepare_sources.py
```

lit les fichiers Excel, normalise les noms de colonnes et génère les fichiers
CSV intermédiaires utilisés par DuckDB.

### Chargement dans DuckDB

Le script :

```text
scripts/01_load_sources.sql
```

charge les sources préparées dans la base locale :

```text
storage/wine_pipeline.duckdb
```

### Nettoyage et déduplication

Les scripts :

```text
scripts/02_web_clean_before_dedup.sql
scripts/03_suppress_duplicates.sql
```

permettent notamment :

- de supprimer les lignes dont les identifiants critiques sont absents ;
- de normaliser les données web ;
- de supprimer les doublons ;
- de conserver une seule ligne pertinente par référence.

### Fusion des systèmes

Le script :

```text
scripts/04_systems_fusion.sql
```

rapproche les tables ERP, liaison et web.

La table consolidée contient :

```text
714 produits rapprochés
```

Elle réunit notamment :

- le prix unitaire ;
- la quantité vendue ;
- le chiffre d’affaires ;
- le stock disponible ;
- les informations descriptives du produit.

## Contrôles qualité

Cinq scripts de contrôle sont intégrés directement au workflow.

| Test | Objectif |
|---|---|
| `test_duplicates_absence.py` | Vérifier l’absence de doublons |
| `test_nulls_absence.py` | Contrôler les valeurs nulles critiques |
| `test_joins_consistency.py` | Vérifier la cohérence et la volumétrie des jointures |
| `test_sales_revenue_consistency.py` | Valider le calcul du chiffre d’affaires |
| `test_zscore.py` | Contrôler le z-score et la classification |

Chaque test est placé immédiatement après la transformation qu’il valide.

Lorsqu’un contrôle critique échoue, le script retourne une erreur et Kestra
interrompt les tâches suivantes. Le pipeline évite ainsi de produire des
exports fondés sur des données incorrectes.

## Calcul du chiffre d’affaires

Le chiffre d’affaires par produit est calculé selon la formule :

```text
Chiffre d’affaires = prix × quantité vendue
```

Le script :

```text
scripts/05_sales_revenue.sql
```

produit :

- le chiffre d’affaires par produit ;
- le chiffre d’affaires global.

Le résultat validé est :

```text
70 568,60 €
```

## Détection des prix atypiques

Le script :

```text
scripts/06_zscore.py
```

calcule le z-score de chaque prix :

```text
z = (prix - moyenne des prix) / écart-type des prix
```

Cette mesure indique la distance entre le prix d’un produit et le prix moyen
du catalogue.

Le z-score est utilisé pour identifier les produits aux prix les plus élevés
et les classer dans deux catégories :

```text
premium
ordinaire
```

Le contrôle dédié vérifie la présence du z-score, sa cohérence et la
volumétrie de chaque catégorie.

## Fichiers générés

Le script :

```text
scripts/07_export_outputs.py
```

génère trois livrables principaux.

### Rapport du chiffre d’affaires

```text
data/output/rapport_chiffre_affaires.xlsx
```

Le fichier Excel contient notamment :

- le chiffre d’affaires par produit ;
- le chiffre d’affaires global.

### Vins premium

```text
data/output/vins_premium.csv
```

Ce fichier contient :

```text
30 vins premium
```

### Vins ordinaires

```text
data/output/vins_ordinaires.csv
```

Ce fichier contient :

```text
684 vins ordinaires
```

Des fichiers CSV intermédiaires sont également produits pendant la préparation
des sources.

## Infrastructure Kestra

L’environnement Kestra est défini dans :

```text
kestra/docker-compose.yml
```

Deux services sont utilisés :

| Service | Rôle |
|---|---|
| `kestra` | Orchestration, planification et supervision |
| `postgres` | Stockage des métadonnées internes de Kestra |

Kestra fonctionne en mode serveur autonome.

Le dépôt est monté dans le conteneur sous :

```text
/app/project
```

L’interface web est accessible localement à l’adresse :

```text
http://localhost:8080
```

## Organisation du dépôt

```text
.
├── data/
│   ├── input/
│   │   └── .gitkeep
│   ├── output/
│   │   └── .gitkeep
│   └── staging/
│       └── .gitkeep
├── flows/
│   └── wine_pipeline.yaml
├── kestra/
│   └── docker-compose.yml
├── presentation/
│   └── projet-10-pipeline-orchestration-flux.pdf
├── scripts/
│   ├── tests/
│   │   ├── test_duplicates_absence.py
│   │   ├── test_joins_consistency.py
│   │   ├── test_nulls_absence.py
│   │   ├── test_sales_revenue_consistency.py
│   │   └── test_zscore.py
│   ├── 00_prepare_sources.py
│   ├── 01_load_sources.sql
│   ├── 02_web_clean_before_dedup.sql
│   ├── 03_suppress_duplicates.sql
│   ├── 04_systems_fusion.sql
│   ├── 05_sales_revenue.sql
│   ├── 06_zscore.py
│   └── 07_export_outputs.py
├── storage/
│   └── .gitkeep
├── .gitignore
└── README.md
```

Les principaux éléments ont les rôles suivants :

| Élément | Rôle |
|---|---|
| `flows/` | Définition du workflow Kestra |
| `scripts/` | Préparation, transformations et exports |
| `scripts/tests/` | Contrôles qualité bloquants |
| `kestra/` | Environnement Docker Compose |
| `presentation/` | Présentation de soutenance |
| `storage/` | Base DuckDB générée localement |
| `data/input/` | Fichiers sources locaux |
| `data/output/` | Exports générés localement |

Les fichiers sources, les sorties et la base DuckDB ne sont pas versionnés.
Les fichiers `.gitkeep` conservent la structure des dossiers vides.

## Résultats

Le pipeline validé produit les résultats suivants :

| Indicateur | Résultat |
|---|---:|
| Références ERP après déduplication | 825 |
| Références web après déduplication | 714 |
| Produits rapprochés | 714 |
| Chiffre d’affaires global | 70 568,60 € |
| Vins premium | 30 |
| Vins ordinaires | 684 |
| Millésimes représentés | 30 |

Le workflow complet s’exécute en treize tâches et bloque les exports lorsqu’un
contrôle critique échoue.

## Livrables

Le projet comprend :

- un workflow Kestra défini dans `flows/wine_pipeline.yaml` ;
- un déclenchement mensuel programmé le 15 de chaque mois à 09:00 ;
- huit scripts de préparation, transformation, calcul et export ;
- cinq scripts de contrôle qualité intégrés au workflow ;
- un environnement Docker Compose exécutant Kestra et PostgreSQL ;
- une base DuckDB locale contenant les différentes étapes de transformation ;
- un rapprochement consolidé de 714 produits entre les systèmes ERP et web ;
- un rapport Excel du chiffre d’affaires par produit et global ;
- un fichier CSV contenant 30 vins premium ;
- un fichier CSV contenant 684 vins ordinaires ;
- une classification des produits fondée sur le z-score ;
- une présentation de soutenance au format PDF ;
- une documentation technique dans le README ;
- un dépôt GitHub public documenté.

## Difficultés rencontrées

### Rapprochement de plusieurs systèmes

Les références ERP et web ne peuvent pas être jointes directement sans le
fichier de liaison.

Une étape de fusion dédiée et un test de cohérence des jointures garantissent
la fiabilité du rapprochement.

### Déduplication

Les doublons peuvent provoquer plusieurs correspondances et fausser le chiffre
d’affaires.

Ils sont supprimés avant la fusion, puis leur absence est validée par un test.

### Qualité des données

Une transformation techniquement réussie peut produire des résultats
fonctionnellement incorrects.

Des contrôles métier sont donc exécutés après chaque étape critique.

### Ordonnancement des tâches

Les tests doivent être placés immédiatement après les transformations qu’ils
contrôlent.

Le workflow empêche les tâches suivantes de s’exécuter lorsqu’un contrôle
échoue.

### Reproductibilité de l’environnement

Kestra et PostgreSQL sont conteneurisés afin de limiter les différences entre
les environnements locaux.

Les données sources restent toutefois nécessaires pour reproduire l’exécution
complète.

## Limites

Le projet constitue une implémentation locale adaptée à un POC.

Ses principales limites sont :

- les fichiers Excel sont ajoutés manuellement ;
- les données sources ne sont pas publiées ;
- la base DuckDB reste locale ;
- les exports ne sont pas historisés ;
- aucune ingestion incrémentale n’est mise en œuvre ;
- aucune notification automatique n’est envoyée en cas d’échec ;
- les métriques d’exécution ne sont pas exportées ;
- les secrets PostgreSQL ne sont pas externalisés ;
- aucun environnement de préproduction n’est fourni ;
- aucun pipeline CI/CD n’est intégré.

## Évolutions possibles

Les évolutions envisagées comprennent :

- remplacer les fichiers Excel par une API ou une source applicative ;
- stocker les sources et les exports dans un stockage objet ;
- historiser les résultats mensuels ;
- ajouter des traitements incrémentaux ;
- intégrer des tests de fraîcheur ;
- ajouter des notifications en cas d’échec ;
- publier des métriques d’exécution ;
- mettre en place une stratégie de reprise ;
- déployer Kestra dans le cloud ;
- utiliser une base analytique centralisée ;
- externaliser les secrets ;
- figer les versions des images Docker ;
- ajouter un environnement de préproduction ;
- mettre en place une chaîne CI/CD.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- conception d’un pipeline de données ;
- orchestration de workflows avec Kestra ;
- planification avec une expression cron ;
- manipulation de données avec Python et Pandas ;
- transformation de données avec SQL ;
- utilisation d’une base analytique DuckDB ;
- nettoyage et normalisation de données ;
- suppression des doublons ;
- rapprochement de plusieurs systèmes ;
- calcul d’indicateurs métier ;
- détection de valeurs atypiques ;
- conception de tests de qualité bloquants ;
- gestion des erreurs dans un workflow ;
- génération de fichiers CSV et Excel ;
- conteneurisation avec Docker ;
- configuration d’un environnement Kestra ;
- documentation d’un pipeline reproductible.
