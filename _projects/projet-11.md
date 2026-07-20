---
title: "Concevez et déployez un système RAG"
subtitle: "Assistant conversationnel de recommandation d’événements culturels"
project_number: 11
order: 11
status: "Terminé"

summary: >-
  Conception d’un système Retrieval-Augmented Generation capable
  de collecter et préparer des événements culturels parisiens,
  de les indexer dans FAISS et de générer des recommandations
  contextualisées avec un modèle Mistral.

domain:
  - Intelligence artificielle
  - Retrieval-Augmented Generation
  - Traitement du langage naturel
  - Recherche vectorielle
  - Qualité des données
  - Data Engineering

technologies:
  - Python
  - LangChain
  - LangChain MistralAI
  - Mistral AI
  - FAISS
  - OpenAgenda
  - Pandas
  - Parquet
  - Pytest
  - Git
  - GitHub

github_url: "https://github.com/ericginez/RAG-chatbot"
---

## Contexte

Puls-Events est une entreprise spécialisée dans la gestion et la
valorisation d’événements culturels.

Elle souhaite proposer un assistant conversationnel capable de recommander
des événements à partir d’une question formulée en langage naturel.

Les informations disponibles proviennent de données publiques OpenAgenda.
Elles doivent être collectées, nettoyées, filtrées et indexées afin que le
chatbot puisse retrouver les événements les plus pertinents avant de
générer une réponse contextualisée.

Le système doit également reconnaître les demandes situées hors de son
périmètre et éviter de produire des recommandations non fondées.

## Présentation

- [Consulter la présentation de soutenance au format PDF](https://github.com/ericginez/RAG-chatbot/blob/main/presentation/projet-11-chatbot-rag.pdf)

## Objectifs

Le projet répond aux objectifs suivants :

- collecter des événements culturels depuis une API publique ;
- limiter la collecte à Paris et aux événements récents ;
- appliquer un filtrage métier sur les spectacles vivants et audiovisuels ;
- nettoyer et normaliser les données récupérées ;
- produire des exports aux formats CSV, JSON et Parquet ;
- transformer les événements en documents textuels exploitables ;
- segmenter les documents en chunks ;
- générer des embeddings avec Mistral ;
- indexer les vecteurs dans FAISS ;
- rechercher les événements les plus proches d’une requête ;
- générer des recommandations en français ;
- encadrer les réponses avec un prompt métier ;
- gérer explicitement les demandes hors périmètre ;
- tester les données, l’environnement et l’index vectoriel ;
- évaluer la qualité fonctionnelle des réponses.

## Périmètre fonctionnel

Le système traite uniquement les événements :

- situés à Paris ;
- datant de moins d’un an ;
- relevant des spectacles vivants ou audiovisuels.

Les catégories retenues comprennent notamment :

```text
Concert
Festival
Théâtre
Danse
Opéra
Cirque
Humour
Stand-up
Performance
Cinéma
Projection
Avant-première
Spectacle
```

Les expositions, musées et autres événements hors périmètre métier ne sont
pas conservés dans le corpus final.

## Source et préparation des données

Le script :

```text
scripts/extract_openagenda.py
```

interroge l’API OpenData Île-de-France utilisée pour exposer les événements
OpenAgenda.

Le traitement applique successivement :

1. un filtrage géographique sur Paris ;
2. un filtrage temporel sur les événements de moins d’un an ;
3. un filtrage métier sur les catégories culturelles retenues ;
4. un nettoyage des champs textuels ;
5. une harmonisation des dates et métadonnées ;
6. la création d’un champ textuel consolidé pour la vectorisation ;
7. l’export des données préparées.

Après les filtres géographique et temporel, le jeu comprend :

```text
6 156 événements
```

Après le filtrage métier, le corpus final contient :

```text
1 336 événements
```

Les données nettoyées sont exportées aux formats :

```text
CSV
JSON
Parquet
```

## Architecture du système RAG

```text
OpenAgenda
     │
     ▼
Extraction et filtrage
     │
     ▼
Nettoyage et consolidation textuelle
     │
     ▼
Chunking
     │
     ▼
Embeddings Mistral
     │
     ▼
Index vectoriel FAISS
     │
     ▼
Question utilisateur
     │
     ▼
Recherche sémantique
     │
     ▼
Top 3 des chunks pertinents
     │
     ▼
Prompt métier + contexte
     │
     ▼
Modèle Mistral
     │
     ▼
Réponse contextualisée
```

Cette architecture dissocie :

- la collecte des données ;
- la préparation documentaire ;
- la recherche vectorielle ;
- la génération de la réponse.

Le modèle génératif reçoit uniquement le contexte sélectionné par la
recherche sémantique.

## Chunking des événements

Le script :

```text
scripts/chunk_events.py
```

charge le fichier JSON nettoyé et utilise
`RecursiveCharacterTextSplitter` pour découper les textes.

Chaque chunk conserve les métadonnées utiles de l’événement :

- titre ;
- lieu ;
- dates ;
- catégorie ;
- URL ;
- informations nécessaires à la restitution.

Le traitement produit :

```text
3 383 chunks
```

Ce découpage réduit la taille des documents transmis à la recherche tout en
préservant suffisamment de contexte pour générer une recommandation utile.

## Vectorisation et index FAISS

Le script :

```text
scripts/vector_store.py
```

calcule les embeddings avec Mistral puis construit l’index FAISS.

Les fichiers produits sont :

```text
data/processed/faiss_index/index.faiss
data/processed/faiss_index/index.pkl
```

L’index contient :

```text
3 383 vecteurs
```

Le nombre de vecteurs correspond exactement au nombre de chunks générés.

FAISS permet ensuite de rechercher rapidement les documents les plus proches
de la représentation vectorielle de la question utilisateur.

## Fonctionnement du chatbot

Le script :

```text
scripts/rag_chatbot.py
```

implémente le chatbot en ligne de commande.

Pour chaque question, le traitement :

1. transforme la requête en représentation vectorielle ;
2. interroge l’index FAISS ;
3. récupère les trois chunks les plus proches ;
4. construit un contexte documentaire ;
5. injecte le contexte et la question dans le prompt ;
6. transmet la requête au modèle Mistral ;
7. affiche la réponse dans le terminal.

Le système vise une réponse contenant deux à quatre recommandations lorsque
des événements pertinents sont disponibles.

## Prompt métier et limitation des hallucinations

Le prompt impose notamment :

- une réponse en français ;
- l’utilisation exclusive du contexte récupéré ;
- une sélection limitée d’événements ;
- la restitution d’informations concrètes ;
- l’absence d’invention lorsque les données sont insuffisantes ;
- un traitement explicite des demandes hors périmètre.

Lorsqu’une question ne concerne pas les événements culturels indexés, la
réponse commence par une indication précisant que la demande est hors du
périmètre du chatbot.

Cette règle empêche le système de se comporter comme un assistant généraliste.

## Tests automatisés

Trois familles de contrôles sont fournies.

### Validation de l’environnement

```text
tests/test_environment.py
```

Ce script vérifie notamment la disponibilité de :

- LangChain ;
- FAISS ;
- NumPy ;
- Mistral SDK.

### Validation des données OpenAgenda

```text
tests/test_openagenda.py
```

Les tests contrôlent la qualité minimale du jeu préparé, notamment :

- la présence des champs essentiels ;
- la cohérence du nombre d’événements ;
- le respect du périmètre géographique et métier ;
- la validité des données nécessaires au RAG.

### Validation de l’index FAISS

```text
tests/test_faiss_index.py
```

Le contrôle vérifie :

- la présence de l’index ;
- la correspondance entre chunks et vecteurs ;
- la cohérence de la recherche sémantique ;
- la restitution des métadonnées.

Les résultats validés sont :

```text
1 336 événements
3 383 chunks
3 383 vecteurs FAISS
```

## Évaluation fonctionnelle

Un jeu de 24 questions a été construit pour évaluer le comportement du
chatbot.

Les catégories comprennent :

| Catégorie | Nombre de cas |
|---|---:|
| Requêtes nominales | 8 |
| Requêtes nécessitant une inférence | 4 |
| Demandes hors périmètre | 4 |
| Risques d’hallucination | 4 |
| Demandes impossibles à satisfaire | 4 |

Exemples de situations testées :

- recherche d’un spectacle sur l’astronomie ;
- demande d’un événement sur l’univers pour des enfants ;
- recherche d’une exposition scientifique hors périmètre ;
- demande d’un concert de Taylor Swift à Paris le lendemain ;
- recherche du meilleur spectacle de l’année suivante.

Le résultat final est :

```text
22 réponses correctes sur 24
Taux de réussite : 91,7 %
```

## Organisation du dépôt

```text
.
├── app/
│   └── dossier réservé à une future interface utilisateur
├── data/
│   ├── raw/
│   └── processed/
│       ├── faiss_index/
│       │   ├── index.faiss
│       │   └── index.pkl
│       ├── openagenda_chunks.json
│       ├── openagenda_paris_clean.csv
│       ├── openagenda_paris_clean.json
│       └── openagenda_paris_clean.parquet
├── presentation/
│   └── projet-11-chatbot-rag.pdf
├── scripts/
│   ├── chunk_events.py
│   ├── extract_openagenda.py
│   ├── rag_chatbot.py
│   └── vector_store.py
├── tests/
│   ├── test_environment.py
│   ├── test_faiss_index.py
│   └── test_openagenda.py
├── .gitignore
├── README.md
└── requirements.txt
```

Les principaux éléments ont les rôles suivants :

| Élément | Rôle |
|---|---|
| `scripts/extract_openagenda.py` | Collecte, filtrage et préparation |
| `scripts/chunk_events.py` | Découpage des événements en chunks |
| `scripts/vector_store.py` | Création de l’index vectoriel |
| `scripts/rag_chatbot.py` | Recherche et génération des réponses |
| `tests/` | Contrôles techniques et qualité |
| `data/processed/` | Données préparées et artefacts RAG |
| `presentation/` | Présentation de soutenance |
| `app/` | Emplacement prévu pour une future interface |

Le fichier `.env` contenant les clés API reste local et n’est pas versionné.

## Résultats

Le projet aboutit à :

- un pipeline d’extraction et de préparation OpenAgenda ;
- un corpus de 1 336 événements parisiens ;
- 3 383 chunks documentaires ;
- un index FAISS de 3 383 vecteurs ;
- une recherche sémantique sur les événements ;
- un chatbot RAG en ligne de commande ;
- des réponses contextualisées en français ;
- une gestion des demandes hors périmètre ;
- des tests automatisés sur les données et l’index ;
- un taux de réussite fonctionnelle de 91,7 %.

Le POC valide la chaîne complète depuis la source publique jusqu’à la
génération de recommandations.

## Livrables

Le projet comprend :

- un script d’extraction et de préparation des données OpenAgenda ;
- des exports nettoyés aux formats CSV, JSON et Parquet ;
- un jeu final de 1 336 événements culturels ;
- un script de génération des chunks ;
- un fichier contenant 3 383 chunks textuels ;
- un script de vectorisation et de création de l’index ;
- un index FAISS contenant 3 383 vecteurs ;
- un chatbot RAG exécutable en ligne de commande ;
- un prompt métier limitant les hallucinations ;
- trois familles de tests automatisés ;
- un protocole d’évaluation de 24 questions ;
- une présentation de soutenance au format PDF ;
- une documentation technique dans le README ;
- un dépôt GitHub public documenté.

## Difficultés rencontrées

### Qualité hétérogène des données

Les événements OpenAgenda présentent des descriptions, formats et niveaux de
détail variables.

Un pipeline de nettoyage et un champ textuel consolidé ont été nécessaires
pour obtenir des documents suffisamment homogènes.

### Définition du périmètre métier

Les catégories OpenAgenda ne correspondent pas toujours directement au
catalogue attendu.

Le filtrage recherche donc plusieurs termes dans les titres, descriptions,
mots-clés et catégories.

### Taille des chunks

Des chunks trop longs réduisent la précision de la recherche, tandis que des
segments trop courts perdent leur contexte.

La stratégie de découpage a été ajustée pour conserver un compromis entre
granularité et cohérence sémantique.

### Utilisation de FAISS sous Windows

FAISS peut rencontrer des limitations avec les chemins synchronisés par
Google Drive en raison de ses appels natifs.

Le contournement consiste à écrire temporairement l’index dans un dossier
local, puis à copier les fichiers vers le projet.

### Contrôle des hallucinations

Le modèle pouvait produire des informations absentes des documents
récupérés.

Le prompt a été renforcé pour imposer l’utilisation exclusive du contexte et
signaler les demandes impossibles ou hors périmètre.

## Limites

Le projet constitue un POC et présente plusieurs limites :

- l’interface est uniquement disponible en ligne de commande ;
- les événements concernent uniquement Paris ;
- le corpus couvre une période limitée ;
- l’actualisation des données doit être relancée manuellement ;
- l’index doit être reconstruit après chaque mise à jour du corpus ;
- les embeddings et les réponses dépendent d’une API externe ;
- aucune authentification utilisateur n’est fournie ;
- aucune mémoire conversationnelle n’est implémentée ;
- le système ne collecte pas encore de retour utilisateur ;
- aucun déploiement cloud n’est fourni ;
- aucun pipeline CI/CD n’est intégré.

## Évolutions possibles

Les évolutions envisagées comprennent :

- améliorer le prompt et les règles de filtrage ;
- ajouter une interface web ;
- automatiser la mise à jour des événements ;
- reconstruire l’index de manière incrémentale ;
- ajouter des filtres explicites sur la date et le lieu ;
- intégrer une mémoire conversationnelle ;
- ajouter un historique des requêtes ;
- recueillir la satisfaction des utilisateurs ;
- enrichir le protocole d’évaluation ;
- comparer plusieurs modèles d’embeddings ;
- comparer plusieurs modèles génératifs ;
- déployer le chatbot dans le cloud ;
- ajouter une API ;
- superviser les performances et les coûts ;
- mettre en place une chaîne CI/CD.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- collecte de données depuis une API ;
- nettoyage et normalisation de données textuelles ;
- préparation d’un corpus documentaire ;
- conception d’une architecture RAG ;
- découpage de documents ;
- génération d’embeddings ;
- utilisation d’une base vectorielle FAISS ;
- intégration d’un modèle Mistral ;
- développement de chaînes LangChain ;
- conception de prompts métier ;
- réduction des hallucinations ;
- tests automatisés de données et d’index ;
- définition d’un protocole d’évaluation ;
- analyse des limites d’un système d’intelligence artificielle ;
- documentation d’un POC Data Engineering.
