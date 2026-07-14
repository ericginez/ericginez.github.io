---
title: "Concevez et déployez un système RAG"
subtitle: "Assistant conversationnel de recommandation d’événements culturels"
project_number: 11
order: 11
status: "Terminé"

summary: >-
  Conception d’un système Retrieval-Augmented Generation capable
  de rechercher des événements culturels pertinents dans une base
  vectorielle FAISS et de générer des recommandations contextualisées
  avec un modèle Mistral.

domain:
  - Intelligence artificielle
  - Retrieval-Augmented Generation
  - Traitement du langage naturel
  - Recherche vectorielle
  - Qualité des données

technologies:
  - Python
  - LangChain
  - Mistral
  - FAISS
  - OpenAgenda
  - Pandas
  - Jupyter

github_url: "https://github.com/ericginez/RAG-chatbot"
---

## Contexte

Une organisation souhaite faciliter la découverte d’événements culturels
à Paris à partir de questions formulées en langage naturel.

Les données disponibles proviennent de l’API OpenAgenda. Elles doivent
être collectées, nettoyées, enrichies et indexées afin qu’un assistant
conversationnel puisse retrouver les événements les plus pertinents et
produire une réponse contextualisée.

Le système doit également reconnaître les questions situées hors de son
périmètre afin d’éviter de générer des réponses non fondées.

## Objectifs

Le projet répond à plusieurs objectifs :

- collecter des événements culturels depuis une API externe ;
- nettoyer et normaliser les données récupérées ;
- transformer les événements en documents exploitables par un système RAG ;
- segmenter les documents en unités adaptées à la recherche sémantique ;
- générer des représentations vectorielles ;
- indexer les vecteurs dans une base FAISS ;
- retrouver les événements pertinents à partir d’une requête utilisateur ;
- générer des recommandations en français avec un modèle Mistral ;
- limiter les hallucinations et gérer les demandes hors périmètre ;
- évaluer la pertinence des réponses produites.

## Périmètre des données

Le système exploite les événements culturels organisés à Paris au cours
des douze derniers mois.

La collecte est centrée sur plusieurs catégories culturelles :

- concerts ;
- festivals ;
- théâtre ;
- danse ;
- opéra ;
- cirque ;
- humour et stand-up ;
- performances ;
- cinéma et projections ;
- avant-premières ;
- spectacles.

Après collecte et préparation, le corpus comprend **1 336 événements**
culturels.

## Architecture du système RAG

Le système repose sur une chaîne de traitement en plusieurs étapes :

1. extraction des événements depuis OpenAgenda ;
2. nettoyage et normalisation des données ;
3. construction de documents textuels structurés ;
4. découpage des documents en chunks ;
5. calcul des embeddings ;
6. indexation des vecteurs dans FAISS ;
7. recherche des chunks les plus proches de la requête ;
8. transmission du contexte au modèle Mistral ;
9. génération d’une réponse en français.

Cette architecture permet de dissocier la base documentaire du modèle
génératif et de fournir au modèle uniquement les informations utiles à
la requête.

## Collecte et préparation

Les données collectées comportent notamment :

- le titre de l’événement ;
- sa description ;
- ses dates ;
- son lieu ;
- sa catégorie ;
- ses conditions d’accès ;
- son URL ;
- ses informations de localisation.

Le pipeline de préparation applique plusieurs traitements :

- suppression des événements incomplets ou inutilisables ;
- normalisation des valeurs textuelles ;
- homogénéisation des dates ;
- regroupement des informations utiles ;
- suppression des contenus redondants ;
- création d’un texte documentaire cohérent pour chaque événement.

## Segmentation et indexation

Les documents sont découpés afin de conserver des unités suffisamment
courtes pour la recherche sémantique, tout en maintenant le contexte
nécessaire à la génération.

Le corpus produit :

- **3 383 chunks documentaires** ;
- **3 383 vecteurs indexés dans FAISS**.

FAISS permet ensuite d’effectuer une recherche de similarité rapide entre
la représentation vectorielle de la question et celles des documents.

## Recherche et génération

Lorsqu’un utilisateur pose une question, le système :

1. transforme la requête en embedding ;
2. interroge l’index FAISS ;
3. sélectionne les documents les plus pertinents ;
4. construit un contexte documentaire ;
5. transmet ce contexte au modèle Mistral ;
6. génère une réponse synthétique.

Le prompt impose notamment :

- une réponse en français ;
- une sélection limitée à deux à quatre événements ;
- des informations concrètes et vérifiables ;
- l’utilisation exclusive du contexte récupéré ;
- une réponse explicite lorsque la demande est hors périmètre.

## Gestion des demandes hors périmètre

Une règle spécifique empêche le chatbot de répondre comme un assistant
généraliste.

Lorsqu’une question ne concerne pas les événements culturels indexés,
la réponse commence par une indication explicite signalant que la demande
est située hors du périmètre du système.

Cette stratégie réduit le risque d’hallucination et clarifie les limites
fonctionnelles de l’assistant.

## Évaluation

Une campagne d’évaluation a été réalisée à partir de plusieurs catégories
de requêtes :

- demandes simples sur un type d’événement ;
- recherches combinant plusieurs critères ;
- requêtes géographiques ;
- demandes avec contraintes temporelles ;
- questions ne correspondant pas au périmètre culturel ;
- formulations ambiguës ou incomplètes.

Le système a validé **20 scénarios sur 24** lors de l’évaluation finale.

Les critères observés portaient notamment sur :

- la pertinence des événements retrouvés ;
- le respect du nombre de recommandations ;
- l’utilisation correcte du contexte ;
- la qualité de la formulation ;
- la gestion des demandes hors périmètre ;
- l’absence d’informations inventées.

## Résultats

La version finale permet :

- d’interroger un corpus de **1 336 événements** ;
- d’effectuer une recherche dans **3 383 vecteurs FAISS** ;
- de proposer entre deux et quatre recommandations ;
- de restituer des réponses contextualisées en français ;
- de filtrer les demandes hors périmètre ;
- de séparer clairement collecte, indexation, recherche et génération.

Le système constitue un prototype fonctionnel de moteur de recommandation
culturelle fondé sur une architecture RAG.

## Difficultés rencontrées

### Qualité hétérogène des données

Les événements OpenAgenda présentent des descriptions, formats et niveaux
de détail variables.

La mise en place d’un pipeline de nettoyage et d’une structure documentaire
commune a permis de rendre les informations plus homogènes avant
l’indexation.

### Taille des documents

Des documents trop longs réduisent la précision de la recherche, tandis
que des segments trop courts perdent leur contexte.

La stratégie de découpage a donc été ajustée afin d’obtenir un compromis
entre granularité et conservation du sens.

### Contrôle des hallucinations

Le modèle génératif pouvait produire des éléments non présents dans les
documents récupérés.

Le prompt a été renforcé afin d’imposer l’utilisation exclusive du contexte
et d’encadrer strictement la structure des réponses.

### Détection du hors périmètre

Certaines requêtes ne concernaient pas la recherche d’événements culturels.

Une consigne explicite et un format de réponse spécifique ont été intégrés
pour signaler ces cas sans tenter de générer une recommandation.

## Compétences démontrées

- collecte de données depuis une API ;
- nettoyage et normalisation de données textuelles ;
- préparation d’un corpus documentaire ;
- conception d’une architecture RAG ;
- découpage et indexation de documents ;
- utilisation d’une base vectorielle FAISS ;
- intégration d’un modèle de langage Mistral ;
- développement de chaînes LangChain ;
- conception de prompts ;
- réduction des hallucinations ;
- définition d’un protocole d’évaluation ;
- analyse des limites d’un système d’intelligence artificielle.