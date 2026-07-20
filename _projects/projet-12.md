---
title: "Créez et automatisez une architecture de données"
subtitle: "Pipeline ELT Bronze–Silver–Gold pour les avantages sportifs"
project_number: 12
order: 12
status: "Terminé"

summary: >-
  Conception et automatisation d’un pipeline ELT intégrant des données RH,
  des mobilités domicile–travail et des activités sportives. L’architecture
  Bronze–Silver–Gold est orchestrée avec Kestra, contrôlée par pytest et
  restituée dans Power BI.

domain:
  - Data Engineering
  - Orchestration
  - Business Intelligence
  - Qualité des données
  - Architecture ELT
  - Traitements incrémentaux

technologies:
  - Python
  - PostgreSQL
  - Kestra
  - Docker
  - Docker Compose
  - Power BI
  - Pytest
  - Poetry
  - Pandas
  - Google Routes API
  - Slack Incoming Webhook
  - Git
  - GitHub

github_url: "https://github.com/ericginez/sports-benefits-data-pipeline"
---

## Contexte

Sport Data Solution développe des solutions de monitoring et d’analyse de
performance. L’entreprise souhaite encourager ses salariés à adopter un mode
de vie actif et à privilégier les mobilités domicile–travail durables.

Elle dispose de plusieurs sources de données :

- des informations RH ;
- des déclarations de pratiques sportives ;
- des activités physiques simulées sur le modèle de Strava ;
- des informations de mobilité domicile–entreprise.

Avant ce projet, aucune chaîne automatisée ne permettait de transformer ces
données en décisions fiables.

Le POC doit donc :

- attribuer équitablement les jours de bien-être ;
- calculer les primes liées aux trajets actifs ;
- valoriser les nouvelles activités par des messages Slack ;
- produire des indicateurs consolidés pour la direction ;
- garantir la traçabilité et la qualité des traitements ;
- alimenter Power BI à partir de tables métier contrôlées.

## Présentation

- [Consulter la présentation de soutenance au format PDF](https://github.com/ericginez/sports-benefits-data-pipeline/blob/main/presentation/projet-12-pipeline-donnees-incitations-sportives.pdf)

## Objectifs

Le projet répond aux objectifs suivants :

- charger et contrôler les données RH et sportives ;
- calculer les distances domicile–entreprise pour les mobilités actives ;
- simuler une année d’activités sportives ;
- générer ensuite de nouveaux lots journaliers ;
- produire des messages Slack sans doublons ;
- séparer données brutes, données détaillées et indicateurs métier ;
- orchestrer les traitements avec Kestra ;
- exécuter les contrôles au moment approprié ;
- reconstruire une couche Gold cohérente ;
- alimenter Power BI uniquement avec des données prêtes à l’emploi.

## Règles métier

### Jours de bien-être

Un salarié ayant réalisé au moins :

```text
15 activités sportives
```

dans la fenêtre annuelle glissante bénéficie de :

```text
5 jours de bien-être
```

Le statut métier distingue :

- les salariés éligibles ;
- les salariés sans sport déclaré ;
- les salariés ayant déclaré un sport mais restant non éligibles.

### Mobilité active

Les distances domicile–entreprise sont calculées pour les salariés utilisant :

- la marche ou le running ;
- le vélo, la trottinette ou un mode assimilé.

Les seuils de conformité sont :

| Mode | Distance maximale |
|---|---:|
| Marche/running | 15 km |
| Vélo/Trottinette/Autres | 25 km |

### Prime sportive

Les salariés dont le trajet actif respecte les règles de conformité
bénéficient d’une prime correspondant à :

```text
5 % du salaire brut annuel
```

Le calcul est réalisé dans la couche Gold, et non dans Power BI.

### Notifications

Chaque nouvelle activité incrémentale entraîne la préparation d’un message de
validation et d’encouragement.

Les messages historiques sont conservés comme déjà livrés. Les messages des
nouveaux lots passent successivement par les états :

```text
PENDING
DELIVERED
```

Une activité ne peut générer qu’un seul message.

## Architecture générale

```text
Fichiers RH et sportifs
        │
        ▼
Chargement Python
        │
        ▼
PostgreSQL Bronze
        │
        ▼
Nettoyage et enrichissement
        │
        ▼
PostgreSQL Silver
        │
        ├── contrôles pytest
        ├── messages Slack
        └── activités historiques et incrémentales
        │
        ▼
Construction Gold
        │
        ▼
PostgreSQL Gold
        │
        ▼
Power BI
```

L’environnement Docker Compose comprend :

- une base PostgreSQL métier ;
- une base PostgreSQL dédiée aux métadonnées Kestra ;
- le serveur Kestra ;
- un conteneur Python utilisé comme runner.

Kestra pilote les scripts et les tests dans l’ordre attendu.

## Architecture Bronze–Silver–Gold

### Bronze

La couche Bronze conserve les données sources et les informations de
traçabilité :

- employés ;
- profils sportifs ;
- activités sportives brutes ;
- messages préparés ;
- identifiants de lots ;
- dates de génération.

### Silver

La couche Silver contient les données détaillées nettoyées et enrichies :

- employés et profils validés ;
- distances domicile–entreprise ;
- activités historiques et incrémentales ;
- statuts de livraison Slack ;
- horodatages de notification.

Silver constitue la source de vérité détaillée utilisée pour reconstruire
les indicateurs.

### Gold

La couche Gold fournit les tables directement consommées par Power BI :

- `gold.employee_sport_benefits` ;
- `gold.monthly_aggregates` ;
- `gold.sports` ;
- `gold.refresh_log` ;
- `gold.annual_activity_window`.

La reconstruction Gold est exécutée de manière transactionnelle. Une
incohérence provoque un rollback complet.

## Sources de données

Le pipeline utilise :

- un fichier Excel contenant les données RH ;
- un fichier Excel contenant les sports déclarés ;
- Google Routes API pour calculer certaines distances domicile–travail ;
- des activités Strava simulées ;
- un Slack Incoming Webhook pour diffuser les notifications.

Les données RH et les secrets restent locaux et ne sont pas publiés dans le
dépôt.

## Pipeline initial

Le flow initial construit l’historique de référence couvrant la période :

```text
1er juillet 2025 au 30 juin 2026
```

Il exécute successivement :

1. le chargement des fichiers RH et sportifs ;
2. la validation des données RH ;
3. le calcul des distances domicile–entreprise ;
4. la validation des distances ;
5. la génération des activités sportives historiques ;
6. la validation de l’historique ;
7. la génération des messages historiques ;
8. la validation des messages ;
9. la construction de Gold ;
10. la réconciliation des indicateurs.

Le script principal de simulation est :

```text
scripts/03_generate_strava.py
```

Il s’appuie sur :

```text
scripts/strava_common.py
```

pour centraliser les règles de simulation des quinze sports.

## Pipeline incrémental

Le flow incrémental traite les nouvelles journées après l’historique initial.

Il est planifié quotidiennement et exécute :

1. la génération du prochain lot d’activités ;
2. la validation du lot ;
3. la préparation des messages Slack ;
4. la validation des messages en attente ;
5. l’envoi des notifications ;
6. la validation de la livraison ;
7. la reconstruction de Gold ;
8. la validation des agrégats.

Le script :

```text
scripts/03_generate_incremental_strava.py
```

produit une activité par sport et par journée incrémentale.

Les identifiants de lots et les dates permettent de suivre précisément
l’origine des enregistrements.

## Notifications Slack

Deux scripts séparent la préparation de l’envoi :

```text
scripts/04_generate_slack_messages.py
scripts/04_send_pending_slack_messages.py
```

Cette séparation permet :

- de contrôler les messages avant envoi ;
- de conserver un statut de livraison ;
- d’éviter les doubles notifications ;
- de relancer uniquement les messages encore en attente ;
- de préserver l’historique.

## Qualité et tests

Les contrôles sont regroupés en cinq familles :

1. complétude et couverture des données ;
2. intégrité référentielle et réconciliation des couches ;
3. conformité aux règles métier ;
4. cohérence temporelle, numérique et traçabilité ;
5. notifications Slack et robustesse applicative.

La version consolidée du dépôt exécute :

```text
59 tests pytest
```

Ces tests représentent plusieurs dizaines de contrôles détaillés sur :

- les données RH ;
- les modes de transport ;
- les distances ;
- les activités initiales et incrémentales ;
- les lots de chargement ;
- les messages Slack ;
- la livraison des notifications ;
- les indicateurs Gold ;
- les réconciliations entre Silver et Gold.

Les tests sont exécutés séquentiellement par Kestra lorsque la base se trouve
dans l’état attendu.

## Construction de la couche Gold

Le script :

```text
scripts/05_build_gold.py
```

calcule notamment :

- la fenêtre annuelle glissante ;
- le nombre d’activités par salarié ;
- l’éligibilité aux jours de bien-être ;
- le nombre de jours attribués ;
- la conformité des trajets actifs ;
- la prime sportive ;
- les agrégats mensuels ;
- les indicateurs par sport ;
- les informations de rafraîchissement.

La fenêtre annuelle est recalculée à partir de la date de la dernière activité
disponible dans Silver.

## Power BI

Power BI se connecte uniquement aux tables Gold en mode Import.

Le rapport comprend trois axes principaux :

- une synthèse générale ;
- l’analyse des activités sportives ;
- l’analyse des trajets domicile–travail.

Les principaux indicateurs sont :

- nombre de salariés ;
- nombre de salariés éligibles ;
- jours de bien-être attribués ;
- trajets actifs conformes ;
- prime sportive annuelle ;
- évolution mensuelle des activités ;
- répartition des sports ;
- distribution des distances ;
- répartition des modes de transport.

Les règles métier sont calculées en amont dans Gold. Power BI utilise des
mesures DAX d’agrégation et ne contient pas de nouvelles colonnes métier
calculées.

## Organisation du dépôt

```text
.
├── data/
│   ├── raw/
│   └── exports/
├── docker/
│   └── python/
│       └── Dockerfile
├── kestra/
│   └── flows/
├── powerbi/
├── presentation/
│   └── projet-12-pipeline-donnees-incitations-sportives.pdf
├── scripts/
│   ├── 01_load_rh_excel.py
│   ├── 02_compute_commute_distance.py
│   ├── 03_generate_incremental_strava.py
│   ├── 03_generate_strava.py
│   ├── 04_generate_slack_messages.py
│   ├── 04_send_pending_slack_messages.py
│   ├── 05_build_gold.py
│   └── strava_common.py
├── sql/
│   └── schema.sql
├── tests/
├── .gitignore
├── docker-compose.yml
├── poetry.lock
├── pyproject.toml
└── README.md
```

Les principaux éléments ont les rôles suivants :

| Élément | Rôle |
|---|---|
| `scripts/` | Chargements, calculs, simulations, notifications et Gold |
| `tests/` | Contrôles d’intégration et de qualité |
| `kestra/flows/` | Définition des flows initial et incrémental |
| `sql/schema.sql` | Initialisation des schémas PostgreSQL |
| `powerbi/` | Rapport et ressources de restitution |
| `presentation/` | Présentation de soutenance |
| `docker-compose.yml` | Orchestration de l’environnement local |

Les données RH, le fichier `.env`, les secrets et les volumes PostgreSQL ne
sont pas versionnés.

## Résultats

La dernière reconstruction validée produit les résultats suivants :

| Indicateur | Résultat |
|---|---:|
| Salariés | 161 |
| Salariés pratiquant un sport | 95 |
| Salariés sans sport déclaré | 66 |
| Sports suivis | 15 |
| Activités historiques Silver | 3 847 |
| Activités dans la fenêtre annuelle | 3 807 |
| Salariés éligibles | 83 |
| Jours de bien-être attribués | 415 |
| Trajets actifs conformes | 68 |
| Agrégats mensuels | 574 |
| Prime sportive annuelle | 172 482,50 € |

Ces valeurs évoluent lorsque de nouveaux lots incrémentaux sont ajoutés.

## Livrables

Le projet comprend :

- un schéma PostgreSQL Bronze–Silver–Gold ;
- un environnement Docker Compose complet ;
- un flow Kestra initial ;
- un flow Kestra incrémental planifié ;
- les scripts Python de chargement, simulation, notification et construction
  des indicateurs ;
- un module partagé pour les règles de simulation ;
- l’intégration de Google Routes API ;
- l’intégration de Slack Incoming Webhook ;
- une suite de tests pytest ;
- cinq tables Gold destinées à la restitution ;
- un rapport Power BI ;
- une présentation de soutenance au format PDF ;
- une documentation technique dans le README ;
- un dépôt GitHub public documenté.

## Difficultés rencontrées

### Cohérence entre historique et incrémental

Les traitements historiques et journaliers doivent appliquer les mêmes règles
métier.

Le module `strava_common.py` évite de dupliquer les paramètres de simulation.

### Gestion des notifications

La génération et l’envoi des messages doivent rester idempotents.

La séparation entre préparation et livraison permet de contrôler les états et
d’éviter les doubles envois.

### Fenêtre annuelle glissante

L’éligibilité dépend d’une période relative à la dernière activité disponible.

La fenêtre est donc recalculée à chaque reconstruction de Gold.

### Réconciliation des couches

Les agrégats Gold doivent correspondre aux données détaillées Silver.

Des tests vérifient les volumes, les périodes, les effectifs, les activités et
les montants calculés.

### Reproductibilité locale

L’environnement comprend plusieurs services et dépend de deux API externes.

Docker Compose, Poetry et les variables d’environnement permettent de
standardiser l’exécution sans publier les secrets.

## Limites

Le projet constitue un POC local et présente plusieurs limites :

- les activités Strava sont simulées ;
- les fichiers RH sont ajoutés manuellement ;
- le calcul des distances dépend d’une API externe ;
- les notifications dépendent d’un webhook Slack ;
- aucun secret manager n’est intégré ;
- l’infrastructure n’est pas déployée dans le cloud ;
- aucune alerte centralisée n’est configurée ;
- aucun pipeline CI/CD n’est fourni ;
- le rapport Power BI nécessite une actualisation après reconstruction de Gold.

## Évolutions possibles

Les évolutions envisagées comprennent :

- connecter une API Strava réelle ;
- remplacer les fichiers RH par une source applicative ;
- déployer l’infrastructure dans le cloud ;
- utiliser un gestionnaire de secrets ;
- ajouter des alertes en cas d’échec ;
- superviser les métriques Kestra ;
- historiser les versions des règles métier ;
- ajouter une stratégie de reprise automatisée ;
- séparer les environnements de développement et de production ;
- déployer une chaîne CI/CD ;
- automatiser l’actualisation du reporting.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- conception d’une architecture ELT ;
- modélisation Bronze, Silver et Gold ;
- développement de traitements Python ;
- gestion de traitements historiques et incrémentaux ;
- orchestration avec Kestra ;
- utilisation de PostgreSQL ;
- intégration d’API externes ;
- conception de traitements idempotents ;
- automatisation de notifications ;
- création de tests d’intégration ;
- réconciliation de couches de données ;
- calcul d’indicateurs métier ;
- conception d’un reporting Power BI ;
- conteneurisation avec Docker ;
- gestion des secrets et de la publication GitHub ;
- documentation d’un pipeline reproductible.
