---
title: "Anticipez les besoins en consommation de bâtiments"
subtitle: "Prédiction énergétique et déploiement d’un modèle avec BentoML"
project_number: 6
order: 6
status: "Terminé"

summary: >-
  Construction d’un modèle de régression destiné à prédire l’intensité
  énergétique normalisée des bâtiments non résidentiels de Seattle.
  Le projet couvre l’analyse exploratoire, le feature engineering,
  la comparaison et l’optimisation de modèles, l’interprétation SHAP
  ainsi que l’exposition du meilleur pipeline avec une API BentoML.

domain:
  - Data Science
  - Machine Learning
  - Data Engineering
  - MLOps
  - API
  - Déploiement cloud

technologies:
  - Python
  - pandas
  - NumPy
  - scikit-learn
  - SHAP
  - BentoML
  - Pydantic
  - Poetry
  - Jupyter Notebook
  - Docker
  - Google Cloud Run
  - Git
  - GitHub

github_url: "https://github.com/ericginez/seattle-building-energy-prediction"
---

## Contexte

La ville de Seattle collecte des données sur les caractéristiques et les
performances énergétiques de ses bâtiments.

La mesure directe de la consommation demande cependant du temps, des
infrastructures et des campagnes de relevés. L’objectif du projet consiste
donc à construire un modèle capable d’estimer la consommation énergétique
d’un bâtiment à partir de caractéristiques disponibles sans mesure annuelle
préalable.

L’étude porte sur les bâtiments non résidentiels de Seattle recensés dans
le jeu de données municipal de benchmarking énergétique de 2016.

La variable à prédire est :

```text
SiteEUIWN(kBtu/sf)
```

Elle représente l’intensité énergétique du site, exprimée en kBtu par pied
carré et normalisée selon les conditions météorologiques.

## Objectifs

Le projet répond aux objectifs suivants :

- analyser et nettoyer un jeu de données énergétique réel ;
- sélectionner les bâtiments non résidentiels exploitables ;
- identifier les variables pertinentes pour la prédiction ;
- prévenir les risques de fuite de données ;
- construire de nouvelles variables explicatives ;
- comparer plusieurs modèles de régression ;
- optimiser le meilleur modèle avec une recherche d’hyperparamètres ;
- interpréter les prédictions avec SHAP ;
- sauvegarder le pipeline complet dans BentoML ;
- exposer le modèle par une API REST ;
- valider les données entrantes avec Pydantic ;
- préparer la containerisation et le déploiement cloud.

## Architecture de la solution

```text
Données énergétiques de Seattle
              |
              v
Analyse exploratoire et nettoyage
              |
              v
Feature engineering
              |
              v
Préprocessing scikit-learn
              |
              v
Comparaison des modèles
              |
              v
Optimisation du Random Forest
              |
              v
Interprétation SHAP
              |
              v
Model Store BentoML
              |
              v
API SeattleEnergyAPI
              |
              v
Conteneur Docker / Cloud Run
```

Le préprocessing et le modèle sont réunis dans un pipeline scikit-learn.
Les mêmes transformations sont donc appliquées pendant l’entraînement,
la validation croisée et l’inférence par l’API.

## Données

Le fichier source contient initialement :

```text
3 376 bâtiments
46 variables
```

Les informations disponibles couvrent notamment :

- l’identification des bâtiments ;
- la localisation géographique ;
- le type et l’usage des propriétés ;
- l’année de construction ;
- les surfaces ;
- le nombre d’étages ;
- les sources d’énergie utilisées ;
- les consommations énergétiques ;
- les émissions de gaz à effet de serre ;
- les indicateurs de conformité.

Le dépôt contient également :

```text
Dictionnary.csv
LargestPropertyUseType.csv
```

Le premier fichier documente les variables du jeu de données. Le second
permet de regrouper les nombreuses modalités d’usage dans des catégories
métier plus homogènes.

## Nettoyage des données

### Sélection des bâtiments

Les bâtiments résidentiels sont retirés de l’étude :

```text
Multifamily LR (1-4)
Multifamily MR (5-9)
Multifamily HR (10+)
```

Le jeu passe alors de 3 376 à 1 668 observations.

Seuls les bâtiments dont le statut de déclaration est `Compliant` sont
ensuite conservés, ce qui réduit le jeu à 1 548 observations.

### Suppression des variables non pertinentes

Les identifiants, adresses, champs textuels libres et variables
administratives sans valeur prédictive sont supprimés.

Cela concerne notamment :

```text
OSEBuildingID
PropertyName
Address
TaxParcelIdentificationNumber
Comments
DataYear
```

Les colonnes présentant une proportion très élevée de valeurs manquantes
sont également retirées, comme :

```text
YearsENERGYSTARCertified
SecondLargestPropertyUseType
ThirdLargestPropertyUseType
ENERGYSTARScore
```

### Traitement des incohérences

Les valeurs nulles de `NumberofBuildings` et `NumberofFloors` sont remplacées
par `1`, car une propriété exploitée possède nécessairement au moins un
bâtiment et un niveau.

Les quelques usages principaux manquants sont reconstruits à partir des
autres informations de la propriété.

Les lignes dont la cible est absente ou égale à zéro sont supprimées.

### Prévention du data leakage

Les variables directement dérivées de la consommation ou des émissions ne
sont pas utilisées comme variables explicatives :

```text
SourceEUIWN(kBtu/sf)
SiteEnergyUseWN(kBtu)
TotalGHGEmissions
GHGEmissionsIntensity
```

Leur conservation aurait fourni au modèle des informations trop directement
liées à la cible.

### Suppression des redondances

La corrélation entre les deux variables de surface suivantes atteint environ
0,95 :

```text
PropertyGFABuilding(s)
LargestPropertyUseTypeGFA
```

Seule la surface totale réelle du bâtiment est conservée :

```text
PropertyGFABuilding(s)
```

## Feature engineering

### Mix énergétique

Les consommations brutes sont converties en trois indicateurs binaires :

```text
UseSteam
UseElectricity
UseNaturalGas
```

Chaque variable indique si la source d’énergie est utilisée ou non.

Cette transformation permet de conserver l’information sur le mix
énergétique sans intégrer directement les volumes de consommation.

### Nombre d’usages

La variable :

```text
NbPropertyUses
```

est calculée à partir du nombre d’usages déclarés dans
`ListOfAllPropertyUseTypes`.

### Âge du bâtiment

La variable `YearBuilt` est remplacée par une information plus directement
interprétable :

```text
BuildingAge = 2016 - YearBuilt
```

L’âge des bâtiments retenus varie de 1 à 116 ans.

### Distance au centre-ville

La distance entre chaque bâtiment et le centre-ville de Seattle est calculée
avec la formule de Haversine :

```text
DistToDowntown_km
```

La distance observée varie approximativement de 0,03 à 14,24 kilomètres.

### Regroupement des usages

La variable `LargestPropertyUseType` contient initialement un grand nombre
de modalités parfois très rares.

Elles sont regroupées dans dix catégories :

```text
Office
Retail
Warehouse
Industrial
Hospitality
Education
Healthcare
Assembly
Public
Other
```

Ce regroupement réduit la cardinalité, limite le risque de surapprentissage
et facilite l’interprétation du modèle.

## Jeu de modélisation

Après l’ensemble des traitements, le jeu final contient :

```text
1 536 observations
14 variables explicatives
1 variable cible
```

### Variables catégorielles

```text
BuildingType
PrimaryPropertyType
LargestPropertyUseType
```

### Variables numériques

```text
Latitude
Longitude
DistToDowntown_km
BuildingAge
NumberofBuildings
NumberofFloors
PropertyGFABuilding(s)
NbPropertyUses
```

### Variables binaires

```text
UseSteam
UseElectricity
UseNaturalGas
```

## Préprocessing

Le préprocessing est intégré dans un `ColumnTransformer`.

```text
Variables catégorielles
    -> OneHotEncoder
    -> gestion des catégories inconnues

Variables numériques
    -> StandardScaler

Variables binaires
    -> conservation directe
```

Le `OneHotEncoder` utilise :

```python
drop="first"
handle_unknown="ignore"
```

Cette configuration évite la redondance entre modalités et permet au modèle
de recevoir une catégorie absente du jeu d’entraînement sans interrompre
l’inférence.

## Comparaison des modèles

Cinq modèles sont comparés par validation croisée à cinq plis.

Les métriques utilisées sont :

- RMSE ;
- MAE ;
- R².

| Modèle | RMSE moyenne | MAE moyenne | R² moyen |
|---|---:|---:|---:|
| Dummy Regressor | 78,64 | 42,99 | -0,084 |
| Régression linéaire | 62,16 | 36,63 | 0,318 |
| SVR | 76,27 | 40,56 | -0,020 |
| Random Forest | 61,23 | 36,94 | 0,336 |
| Gradient Boosting | 61,47 | 36,30 | 0,327 |

Le Random Forest obtient le meilleur R² avant optimisation.

Une transformation logarithmique de la cible est également testée :

| Modèle | RMSE moyenne | MAE moyenne | R² moyen |
|---|---:|---:|---:|
| Random Forest avec cible logarithmique | 62,72 | 33,69 | 0,307 |

La transformation améliore la MAE, mais dégrade le R² et la RMSE. Elle
n’est donc pas retenue pour le modèle final.

## Optimisation du modèle

Une première recherche teste trois hyperparamètres :

```text
n_estimators
max_depth
min_samples_leaf
```

Elle porte le R² moyen à environ :

```text
0,371
```

Une seconde grille étendue teste cinq hyperparamètres et 288 combinaisons :

```text
n_estimators
max_depth
min_samples_leaf
min_samples_split
max_features
```

Les meilleurs paramètres sont :

```python
{
    "model__max_depth": None,
    "model__max_features": "sqrt",
    "model__min_samples_leaf": 1,
    "model__min_samples_split": 2,
    "model__n_estimators": 300
}
```

Le meilleur score obtenu est :

```text
R² moyen = 0,3833
```

L’optimisation apporte environ 4,7 points de R² par rapport au Random Forest
initial.

## Interprétation avec SHAP

Le meilleur modèle est analysé avec `SHAP TreeExplainer`.

L’interprétation met principalement en évidence :

- le type d’activité exercée dans le bâtiment ;
- le mix énergétique utilisé ;
- la surface du bâtiment ;
- son âge ;
- sa localisation ;
- sa structure ;
- le nombre d’usages déclarés.

Les bâtiments industriels, hospitaliers, commerciaux ou spécialisés
présentent des profils énergétiques sensiblement différents des bureaux et
entrepôts classiques.

SHAP permet de dépasser la seule mesure de performance et de comprendre les
facteurs qui orientent les prédictions du modèle.

## Sauvegarde du modèle

Le pipeline complet est sauvegardé dans le Model Store BentoML.

```text
seattle_energy_rf:6iuuoyyzhwbgekgq
```

Le modèle enregistré inclut :

- le préprocessing ;
- l’encodage des variables catégorielles ;
- la standardisation ;
- le Random Forest optimisé ;
- les métadonnées du projet ;
- la liste des variables d’entrée ;
- les meilleurs hyperparamètres.

Environnement du modèle :

```text
Python        : 3.14.0
scikit-learn  : 1.8.0
BentoML       : 1.4.35
Taille        : environ 40 MiB
```

## API BentoML

Le modèle est exposé par le service :

```text
SeattleEnergyAPI
```

Le pipeline est chargé une seule fois au démarrage du service afin d’éviter
un rechargement à chaque requête.

### Endpoints applicatifs

| Endpoint | Fonction |
|---|---|
| `POST /predict` | Prédire l’intensité énergétique |
| `GET /health` | Vérifier l’état applicatif |
| `GET /metadata` | Consulter les métadonnées du service |
| `GET /model-info` | Identifier le pipeline et le modèle chargés |
| `GET /features` | Obtenir la liste des variables attendues |

BentoML expose également ses endpoints techniques :

```text
/readyz
/livez
/healthz
```

### Contrat d’entrée

L’endpoint `/predict` attend quatorze champs :

```text
BuildingType
PrimaryPropertyType
LargestPropertyUseType
Latitude
Longitude
DistToDowntown_km
BuildingAge
NumberofBuildings
NumberofFloors
PropertyGFABuilding(s)
NbPropertyUses
UseSteam
UseElectricity
UseNaturalGas
```

### Validation Pydantic

Les données entrantes sont contrôlées avant leur transmission au modèle.

La validation porte sur :

- les types de données ;
- les champs obligatoires ;
- les bornes numériques ;
- les indicateurs binaires ;
- les longueurs minimales ;
- les noms de champs ;
- les propriétés inattendues.

Les champs supplémentaires sont interdits avec :

```python
extra="forbid"
```

L’alias suivant garantit la compatibilité entre le JSON et le nom de colonne
utilisé pendant l’entraînement :

```text
PropertyGFABuilding(s)
```

### Exemple de requête

```powershell
$payload = @{
    BuildingType             = "NonResidential"
    PrimaryPropertyType      = "Large Office"
    LargestPropertyUseType   = "Office"
    Latitude                 = 47.61
    Longitude                = -122.33
    DistToDowntown_km        = 0.5
    BuildingAge              = 17
    NumberofBuildings        = 1
    NumberofFloors           = 10
    "PropertyGFABuilding(s)" = 250000
    NbPropertyUses           = 1
    UseSteam                 = 0
    UseElectricity           = 1
    UseNaturalGas            = 0
}

Invoke-RestMethod `
    -Uri "http://localhost:3000/predict" `
    -Method Post `
    -ContentType "application/json" `
    -Body ($payload | ConvertTo-Json)
```

### Exemple de réponse

```json
{
  "SiteEUIWN_kBtu_per_sf": 67.25
}
```

## Packaging BentoML

Le projet possède deux environnements complémentaires.

### Environnement de développement

La racine contient :

- le notebook ;
- les données ;
- les bibliothèques d’analyse ;
- SHAP ;
- le code de l’API ;
- la configuration du serveur.

### Environnement de déploiement

Le dossier `app/` constitue un contexte de build autonome contenant :

- les dépendances runtime minimales ;
- le service BentoML ;
- les schémas Pydantic ;
- le fichier `bentofile.yaml` ;
- le tag du modèle validé.

Le Bento est construit avec :

```powershell
Set-Location app
poetry install
poetry run bentoml build
```

Le fichier `bentofile.yaml` inclut explicitement :

```text
src/
pyproject.toml
poetry.lock
README.md
seattle_energy_rf:6iuuoyyzhwbgekgq
```

## Containerisation

Après la construction du Bento, une image Docker peut être produite avec :

```powershell
poetry run bentoml containerize <BENTO>:<TAG> `
    -t seattle-energy-api:latest
```

Le conteneur peut être exécuté localement :

```powershell
docker run --rm -p 3000:3000 seattle-energy-api:latest
```

La disponibilité du service est ensuite vérifiée avec :

```text
http://localhost:3000/readyz
```

## Préparation au déploiement cloud

L’application est préparée pour un déploiement sur Google Cloud Run.

Le processus envisagé est :

```text
Construction du Bento
        |
        v
Création de l’image Docker
        |
        v
Publication dans Artifact Registry
        |
        v
Déploiement dans Cloud Run
        |
        v
Validation de /readyz et /predict
```

Le dépôt contient la configuration nécessaire au packaging et à la
containerisation, mais pas encore de pipeline CI/CD automatisé.

## Résultats

Le projet aboutit à :

- un jeu de données énergétique nettoyé ;
- une sélection de 1 536 bâtiments non résidentiels ;
- quatorze variables explicatives ;
- un pipeline complet de préprocessing ;
- une comparaison de cinq modèles ;
- un Random Forest optimisé ;
- un R² moyen de 0,3833 ;
- une interprétation des prédictions avec SHAP ;
- un modèle versionné dans BentoML ;
- une API REST validée avec Pydantic ;
- cinq endpoints applicatifs ;
- un contexte de build autonome ;
- une préparation à la containerisation et au déploiement cloud.

## Difficultés rencontrées

### Risque de fuite de données

Plusieurs variables énergétiques et d’émissions étaient directement liées à
la cible.

Elles ont été identifiées et retirées afin d’éviter des performances
artificiellement élevées.

### Forte cardinalité des usages

La variable d’usage principal contenait de nombreuses catégories rares.

Un mapping métier a permis de les regrouper dans dix familles plus robustes.

### Distribution asymétrique de la cible

La cible présente une forte asymétrie et des valeurs élevées.

Une transformation logarithmique a été testée, mais n’a pas amélioré les
performances globales du modèle.

### Synchronisation du code de déploiement

Le code est présent dans `src/` pour le développement et dans `app/src/`
pour le packaging autonome.

Cette duplication facilite le déploiement, mais exige de maintenir les deux
copies synchronisées.

### Versionnement du modèle

Le modèle BentoML est stocké dans un Model Store local et non directement
dans le dépôt Git.

Sa reconstruction nécessite de réexécuter le notebook puis de mettre à jour
le nouveau tag du modèle.

## Limites

Le projet présente plusieurs limites :

- les données concernent uniquement Seattle en 2016 ;
- le modèle n’est pas directement généralisable à une autre ville ;
- le R² final reste limité à environ 0,383 ;
- une part importante de la variabilité énergétique reste inexpliquée ;
- certaines catégories disposent de peu d’observations ;
- le modèle BentoML n’est pas stocké dans le dépôt ;
- aucun test automatisé de l’API n’est encore fourni ;
- aucun pipeline CI/CD n’est intégré ;
- le déploiement Cloud Run est préparé mais non automatisé.

## Évolutions possibles

Les évolutions envisagées comprennent :

- tester XGBoost, LightGBM ou CatBoost ;
- ajouter une validation spatiale ;
- comparer plusieurs années de données ;
- ajouter des tests unitaires Pydantic ;
- ajouter des tests d’intégration sur `/predict` ;
- automatiser la synchronisation de `src/` et `app/src/` ;
- stocker le modèle dans un registre distant ;
- ajouter une détection de dérive ;
- exposer des métriques Prometheus ;
- intégrer OpenTelemetry ;
- automatiser le build Docker ;
- créer un pipeline GitHub Actions ;
- automatiser le déploiement Cloud Run.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- analyse exploratoire de données ;
- nettoyage et préparation de données ;
- traitement des valeurs manquantes ;
- prévention du data leakage ;
- feature engineering ;
- réduction de cardinalité ;
- encodage catégoriel ;
- standardisation ;
- pipeline scikit-learn ;
- validation croisée ;
- comparaison de modèles ;
- optimisation d’hyperparamètres ;
- interprétation SHAP ;
- gestion d’un Model Store ;
- développement d’une API de Machine Learning ;
- validation de données avec Pydantic ;
- packaging avec BentoML ;
- containerisation Docker ;
- préparation au déploiement cloud.