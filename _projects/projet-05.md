---
title: "Maintenez et documentez un système de stockage des données sécurisé et performant"
subtitle: "Migration reproductible d’un dataset healthcare vers MongoDB"
project_number: 5
order: 5
status: "Terminé"

summary: >-
  Conception d’une chaîne de migration de données healthcare depuis un fichier
  CSV vers MongoDB. La solution assure le typage des données, l’insertion par
  lots, l’indexation, l’authentification de la base, les contrôles d’intégrité,
  les tests unitaires et l’exécution reproductible avec Docker Compose.

domain:
  - Data Engineering
  - Base de données
  - NoSQL
  - Qualité des données
  - Sécurité
  - DevOps

technologies:
  - Python
  - MongoDB
  - PyMongo
  - Docker
  - Docker Compose
  - GitHub Actions
  - unittest
  - Jupyter Notebook
  - Git
  - GitHub

github_url: "https://github.com/ericginez/healthcare-mongodb-migration"
---

## Contexte

Une organisation dispose d’un jeu de données healthcare stocké dans un
fichier CSV. Ce format est adapté à l’échange de données, mais il devient
moins pratique lorsque les données doivent être interrogées régulièrement,
indexées, contrôlées et exploitées par plusieurs applications.

L’objectif du projet est de migrer ces données vers MongoDB en construisant
une solution :

- reproductible ;
- performante ;
- sécurisée ;
- testable ;
- documentée ;
- exploitable localement avec Docker.

Le projet ne se limite pas à l’import du fichier. Il comprend également la
conversion des types, la création d’index, l’identification des anomalies de
qualité et l’automatisation des tests unitaires avec GitHub Actions.

## Objectifs

La solution répond aux objectifs suivants :

- migrer un fichier CSV vers une collection MongoDB ;
- lire les données sans charger l’intégralité du fichier en mémoire ;
- convertir les chaînes de caractères vers des types adaptés ;
- insérer les documents par lots ;
- sécuriser l’accès à MongoDB par authentification ;
- conserver les données dans un volume persistant ;
- créer des index adaptés aux requêtes ;
- détecter les valeurs manquantes ;
- contrôler les types BSON enregistrés ;
- détecter les doublons métier ;
- appliquer des règles de cohérence ;
- tester automatiquement la logique de conversion ;
- fournir un environnement Docker reproductible ;
- intégrer les tests dans un workflow CI.

## Architecture de la solution

```text
healthcare_dataset.csv
          |
          v
Lecture CSV en streaming
          |
          v
Conversion et normalisation des types
          |
          v
Regroupement en lots de 1 000 documents
          |
          v
Insertion MongoDB avec insert_many()
          |
          v
Création des index
          |
          v
Contrôles d’intégrité
          |
          v
Rapport de qualité des données
```

L’environnement Docker Compose repose sur trois services.

### MongoDB

Le service `mongodb` fournit :

- une instance MongoDB 7 ;
- une authentification par identifiant et mot de passe ;
- un healthcheck ;
- un port accessible localement ;
- un volume Docker persistant.

### Migration

Le service `migrate_healthcare` :

- attend que MongoDB soit disponible ;
- lit le fichier CSV ;
- convertit les valeurs ;
- insère les documents par lots ;
- crée les index ;
- s’arrête automatiquement lorsque la migration est terminée.

### Contrôles d’intégrité

Le service `integrity_checks` :

- attend la fin de la migration ;
- interroge la collection MongoDB ;
- exécute les contrôles de qualité ;
- produit un rapport ;
- ne modifie jamais les données.

## Organisation du dépôt

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── data/
│   └── healthcare_dataset.csv
├── migration/
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── import_healthcare_dataset_to_mongodb.py
│   ├── integrity_checks_mongodb.py
│   ├── requirements.txt
│   └── __init__.py
├── notebooks/
│   ├── import_healthcare_dataset_to_mongodb.ipynb
│   ├── integrity_checks_mongodb.ipynb
│   └── unit_tests.ipynb
├── reports/
│   └── integrity_report.txt
├── unit_tests/
│   ├── test_convert_type.py
│   ├── test_smoke_import_module.py
│   └── __init__.py
├── .env.example
├── .gitattributes
├── .gitignore
├── Checklist_securite_local.md
├── docker-compose.yml
└── README.md
```

## Données

Le fichier source contient :

```text
55 500 lignes
16 colonnes
```

Chaque ligne devient un document de la collection :

```text
healthcare.patients
```

Les données comprennent notamment :

- le nom du patient ;
- l’âge ;
- le genre ;
- le groupe sanguin ;
- la condition médicale ;
- la date d’admission ;
- le médecin ;
- l’hôpital ;
- l’assureur ;
- le montant de facturation ;
- le numéro de chambre ;
- le type d’admission ;
- la date de sortie ;
- le médicament ;
- le résultat du test.

## Migration des données

### Lecture en streaming

Le fichier CSV est lu avec `csv.DictReader`.

La lecture est effectuée ligne par ligne au lieu de charger les 55 500 lignes
dans un DataFrame ou une liste complète.

Cette approche permet :

- de limiter la consommation mémoire ;
- de traiter un fichier plus volumineux ;
- de commencer l’import avant la lecture complète ;
- d’alimenter directement la logique d’insertion par lots.

L’encodage `utf-8-sig` permet également de gérer les fichiers contenant un
BOM UTF-8.

### Conversion des types

La fonction `convert_type()` centralise les règles de conversion.

| Champ | Type enregistré |
|---|---|
| `Age` | entier |
| `Billing Amount` | nombre flottant |
| `Room Number` | entier |
| `Date of Admission` | date BSON |
| `Discharge Date` | date BSON |
| `Name` | chaîne normalisée |
| Autres champs | chaîne nettoyée |
| Valeur vide | `null` |

Les dates sont converties depuis le format :

```text
YYYY-MM-DD
```

Les valeurs textuelles sont nettoyées avec suppression des espaces
superflus.

La centralisation de ces règles garantit que la même logique est appliquée à
toutes les lignes et qu’elle peut être testée indépendamment de MongoDB.

### Insertion par lots

Les documents sont regroupés par lots de :

```text
1 000 documents
```

Chaque lot est inséré avec :

```python
collection.insert_many(batch, ordered=False)
```

Cette stratégie est plus performante qu’une succession d’appels
`insert_one()`.

L’option `ordered=False` demande à MongoDB de poursuivre l’insertion des
autres documents du lot lorsqu’une erreur isolée survient.

### Modes d’import

Le script prend en charge plusieurs modes :

- `--drop` : supprime puis recrée la collection ;
- `--clear` : supprime les documents en conservant les index ;
- aucun argument : ajoute les nouveaux documents ;
- `--skip-indexes` : empêche la recréation des index.

Le projet utilise le mode `--drop` pour assurer une migration complète et
reproductible.

## Index MongoDB

Les index sont créés après l’import, afin de ne pas ralentir chaque
insertion.

### Index simples

```text
Age
Billing Amount
Room Number
Date of Admission
Discharge Date
```

### Index composé

```text
Name + Date of Admission
```

Cet index facilite :

- la recherche d’un patient à une date donnée ;
- le regroupement par clé métier ;
- la détection des doublons ;
- certaines opérations d’agrégation.

L’index composé n’est pas défini comme unique, car les données sources
contiennent des doublons.

## Sécurité

### Authentification MongoDB

MongoDB est démarré avec une authentification administrateur.

Les informations de connexion sont transmises par variables
d’environnement :

```text
MONGO_INITDB_ROOT_USERNAME
MONGO_INITDB_ROOT_PASSWORD
MONGO_URI
MONGO_DB
MONGO_COLLECTION
CSV_PATH
```

### Protection des secrets

Le fichier réel `.env` est exclu du dépôt Git.

Le dépôt fournit uniquement :

```text
.env.example
```

Les valeurs contenues dans ce fichier sont fictives et doivent être
remplacées localement.

### Persistance

Les données MongoDB sont stockées dans un volume Docker :

```text
mongodb_data
```

Elles sont ainsi conservées lorsque les conteneurs sont arrêtés ou recréés.

### Montage en lecture seule

Le répertoire contenant le CSV est monté dans le conteneur de migration en
lecture seule :

```yaml
volumes:
  - ./data:/data:ro
```

Le conteneur peut lire le dataset, mais ne peut pas modifier le fichier
source.

### Healthcheck

Un healthcheck authentifié exécute une commande `ping` sur MongoDB.

Les services de migration ne démarrent qu’après le passage de la base dans
l’état :

```text
healthy
```

Le script Python effectue également un `ping` au démarrage afin d’échouer
rapidement lorsque la base est inaccessible.

## Contrôles d’intégrité

Les contrôles sont exécutés après la migration et n’altèrent pas la
collection.

### Schéma observé

Le script analyse un échantillon de documents afin d’identifier les champs
présents.

Résultat :

```text
16 champs observés
```

### Valeurs manquantes

Les champs suivants sont vérifiés :

```text
Name
Age
Billing Amount
Date of Admission
Discharge Date
```

Un champ est considéré comme manquant lorsqu’il est absent ou égal à
`null`.

Résultat :

```text
Aucune valeur manquante détectée
```

### Contrôle des types

Les types MongoDB attendus sont contrôlés pour :

```text
Age                -> int
Billing Amount     -> double
Date of Admission  -> date
Discharge Date     -> date
```

Résultat :

```text
Aucune anomalie de type détectée
```

### Détection des doublons métier

La clé métier utilisée est :

```text
Name + Date of Admission
```

Résultat :

```text
5 509 groupes de doublons
```

Ce résultat représente un nombre de groupes présentant plusieurs documents,
et non directement un nombre de lignes en trop.

### Règles de cohérence

Trois règles sont appliquées :

| Règle | Anomalies |
|---|---:|
| Âge inférieur à 0 ou supérieur à 120 | 0 |
| Montant de facturation négatif | 108 |
| Date de sortie antérieure à l’admission | 0 |

Les contrôles ont donc révélé :

```text
108 montants de facturation négatifs
```

Ces anomalies sont signalées, mais ne sont pas corrigées automatiquement.

## Résultats du rapport

```text
Documents : 55500
Champs observés : 16
Valeurs manquantes : 0
Anomalies de type : 0
Groupes de doublons : 5509
Âges hors limites : 0
Montants négatifs : 108
Sorties antérieures aux admissions : 0
```

Le rapport est conservé dans :

```text
reports/integrity_report.txt
```

## Tests unitaires

Les tests utilisent le module standard Python :

```text
unittest
```

### Tests de conversion

Les cas testés comprennent :

- conversion d’un âge vers un entier ;
- suppression des espaces ;
- gestion d’une valeur vide ;
- gestion de `None` ;
- rejet d’un âge non numérique ;
- conversion d’un montant vers un flottant ;
- rejet d’un montant au format invalide ;
- conversion d’une date ISO vers `datetime` ;
- rejet d’un format de date invalide ;
- normalisation de la casse d’un nom.

### Smoke test

Un test de fumée vérifie que le module de migration peut être importé sans
erreur.

Il permet notamment de détecter :

- une erreur de syntaxe ;
- une dépendance absente ;
- un mauvais chemin de package ;
- une erreur exécutée au chargement du module.

### Résultat

```text
11 tests exécutés
11 tests réussis
```

## Intégration continue

Le workflow GitHub Actions est exécuté lors de chaque :

```text
push
pull_request
```

La CI teste le projet avec :

```text
Python 3.11
Python 3.12
```

Le workflow :

1. récupère le dépôt ;
2. installe Python ;
3. met à jour `pip` ;
4. installe `pymongo` ;
5. découvre les tests dans `unit_tests/` ;
6. exécute les tests en mode verbeux.

Les contrôles d’intégrité ne sont pas exécutés dans cette CI, car ils
nécessitent une instance MongoDB et les données migrées.

## Exécution locale

### Préparer la configuration

```powershell
Copy-Item .env.example .env
```

Les identifiants du fichier `.env` doivent ensuite être remplacés.

### Démarrer MongoDB

```powershell
docker compose --profile db up -d --wait mongodb
```

### Exécuter la migration

```powershell
docker compose `
    --profile migrate `
    run `
    --rm `
    --no-deps `
    migrate_healthcare
```

### Exécuter les contrôles d’intégrité

```powershell
docker compose `
    --profile integrity `
    run `
    --rm `
    --no-deps `
    integrity_checks
```

### Exécuter les tests unitaires

```powershell
python -m unittest discover `
    -s unit_tests `
    -p "test_*.py" `
    -v
```

### Arrêter l’environnement

```powershell
docker compose down --remove-orphans
```

Le volume MongoDB est conservé par défaut.

## Résultats obtenus

Le projet aboutit à :

- une migration complète de 55 500 lignes ;
- une lecture CSV en streaming ;
- une insertion par lots de 1 000 documents ;
- une collection MongoDB typée ;
- cinq index simples ;
- un index composé ;
- une instance MongoDB authentifiée ;
- un stockage persistant ;
- un healthcheck applicatif ;
- un montage du dataset en lecture seule ;
- un rapport automatisé de qualité ;
- onze tests unitaires réussis ;
- une CI compatible Python 3.11 et 3.12 ;
- un environnement reproductible avec Docker Compose.

## Difficultés rencontrées

### Gestion des types

Un fichier CSV ne conserve pas les types métier. Toutes les valeurs sont
initialement lues sous forme de chaînes.

La solution consiste à centraliser les conversions dans une fonction unique
et testable avant l’insertion dans MongoDB.

### Performance de l’import

Une insertion document par document aurait multiplié les échanges avec la
base.

Le regroupement en lots et l’utilisation de `insert_many()` ont permis de
réduire ces échanges.

### Coordination des services Docker

La migration ne peut commencer qu’après l’initialisation complète de
MongoDB.

Un healthcheck et des dépendances conditionnelles garantissent l’ordre
d’exécution.

### Différence entre succès technique et qualité des données

La migration peut réussir techniquement tout en important des anomalies
présentes dans la source.

Le projet distingue donc :

- les erreurs techniques bloquantes ;
- les anomalies de qualité signalées dans le rapport.

### Gestion des secrets

Les identifiants MongoDB ne doivent jamais être enregistrés dans Git.

Le fichier `.env` est exclu, tandis qu’un modèle `.env.example` documente
les variables nécessaires.

## Limites

Le projet présente plusieurs limites :

- les doublons sont détectés mais non supprimés ;
- les montants négatifs sont signalés mais non corrigés ;
- les contrôles d’intégrité sont informatifs par défaut ;
- la CI ne démarre pas de service MongoDB ;
- aucun contrôle de performance des index n’est automatisé ;
- aucun mécanisme de reprise sur incident n’est implémenté ;
- aucune stratégie de sauvegarde MongoDB n’est automatisée ;
- les droits MongoDB utilisent un compte administrateur pour le POC ;
- le schéma reste flexible et n’utilise pas de validation JSON Schema côté
  MongoDB.

## Évolutions possibles

Les évolutions envisagées comprennent :

- ajouter un rôle MongoDB limité à la base healthcare ;
- définir une validation JSON Schema ;
- créer une collection de quarantaine ;
- journaliser les lignes rejetées ;
- dédupliquer les données avant insertion ;
- ajouter une option d’upsert ;
- rendre les doublons bloquants ;
- rendre certaines anomalies de cohérence bloquantes ;
- tester les index avec `explain()` ;
- intégrer MongoDB dans la CI ;
- produire le rapport au format JSON ;
- archiver les rapports comme artefacts GitHub Actions ;
- ajouter une politique de sauvegarde et de restauration ;
- superviser MongoDB avec des métriques ;
- exécuter la migration de façon incrémentale.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- modélisation de documents MongoDB ;
- migration de données ;
- lecture de fichiers en streaming ;
- typage des données ;
- traitement par lots ;
- indexation NoSQL ;
- contrôle de qualité ;
- détection de doublons ;
- validation de règles métier ;
- sécurisation des secrets ;
- authentification MongoDB ;
- gestion de volumes Docker ;
- orchestration Docker Compose ;
- développement de scripts CLI ;
- tests unitaires Python ;
- intégration continue ;
- documentation technique ;
- versionnement Git.