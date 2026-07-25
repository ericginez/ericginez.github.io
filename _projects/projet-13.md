---
title: "Passez votre système IA du POC au MVP"
subtitle: "Conception et pilotage de l’industrialisation d’un système RAG"
project_number: 13
order: 13
status: "Terminé"

summary: >-
  Définition de la trajectoire permettant de transformer le POC RAG
  Puls-Events en un MVP cloud scalable, observable et sécurisé,
  intégrant mémoire conversationnelle, géolocalisation et recherche
  web en temps réel.

domain:
  - Gestion de projet Data
  - Architecture cloud
  - Intelligence artificielle
  - Retrieval-Augmented Generation
  - FinOps
  - Observabilité

technologies:
  - Jekyll
  - GitHub Pages
  - GitHub
  - Docker
  - Python
  - LangChain
  - FAISS
  - Mistral

github_url: "https://github.com/ericginez/rag-poc-to-mvp-project"
---

## Contexte

Le projet prolonge le système RAG développé pour Puls-Events dans le
Projet 11.

Le POC permet de rechercher des événements culturels dans un corpus issu
d’OpenAgenda et de produire des recommandations avec une architecture
reposant sur LangChain, Mistral et une base vectorielle FAISS.

Cette première version a démontré la faisabilité technique du système,
mais elle reste exécutée localement et comporte plusieurs limites pour
un usage réel :

- chemins de fichiers codés en dur ;
- index vectoriel stocké localement ;
- absence de mémoire conversationnelle persistante ;
- prise en compte limitée de la localisation ;
- absence de recherche web en temps réel ;
- observabilité insuffisante ;
- architecture non conçue pour supporter une montée en charge.

L’objectif du Projet 13 est de définir les conditions permettant de
transformer ce POC en un MVP exploitable, mesurable et évolutif.

## Présentation

- [Consulter la présentation de soutenance au format PDF](https://github.com/ericginez/rag-poc-to-mvp-project/blob/main/presentation/projet-13-rag-poc-to-mvp-project.pdf)

## Objectifs

Le projet poursuit plusieurs objectifs complémentaires :

- recueillir et formaliser les besoins métier ;
- analyser les limites du POC existant ;
- définir le périmètre fonctionnel du MVP ;
- concevoir une architecture cloud scalable ;
- planifier les travaux nécessaires à l’industrialisation ;
- prioriser les fonctionnalités dans un macro-backlog ;
- identifier les risques techniques, organisationnels et budgétaires ;
- estimer les coûts de construction et d’exploitation ;
- définir les modalités de suivi de la qualité et de la performance ;
- présenter les compétences acquises dans un portfolio professionnel.

## POC de référence

Le POC du Projet 11 repose sur la chaîne suivante :

1. collecte des événements depuis OpenAgenda ;
2. nettoyage et préparation des données ;
3. transformation des événements en documents ;
4. segmentation des documents ;
5. génération des embeddings ;
6. indexation dans FAISS ;
7. recherche sémantique ;
8. génération des réponses avec Mistral.

Les principaux résultats obtenus sont :

- **1 336 événements culturels** collectés ;
- **3 383 segments documentaires** générés ;
- **3 383 vecteurs** indexés dans FAISS ;
- **22 réponses conformes sur 24, soit 91,7 %** lors de l’évaluation finale.

Ces résultats confirment la faisabilité du concept, mais ne suffisent pas
à garantir son exploitation dans un environnement de production.

## Besoins fonctionnels du MVP

Quatre évolutions principales structurent le passage au MVP.

### Mémoire conversationnelle

Le système doit conserver les échanges précédents afin de comprendre les
références formulées par l’utilisateur et de proposer des réponses plus
cohérentes au fil de la conversation.

La mémoire doit être stockée indépendamment de l’instance applicative afin
de rester disponible lors d’un redémarrage ou d’une montée en charge.

### Contexte géographique

Les recommandations doivent prendre en compte la localisation de
l’utilisateur, la zone géographique recherchée et la distance acceptable
jusqu’aux événements proposés.

Cette fonctionnalité permet d’améliorer la pertinence des résultats sans
modifier la logique principale du moteur RAG.

### Recherche web en temps réel

Le système doit pouvoir compléter les informations indexées avec des
données disponibles en ligne lorsque celles-ci sont absentes, incomplètes
ou devenues obsolètes.

Cette capacité doit rester encadrée afin de contrôler la qualité des
sources, les quotas d’API, les temps de réponse et les coûts.

### Monitoring de performance

Le MVP doit exposer des indicateurs permettant de suivre :

- les temps de réponse ;
- les erreurs applicatives ;
- la disponibilité du service ;
- le nombre de requêtes ;
- les consommations des services externes ;
- la qualité des résultats produits ;
- les coûts d’exploitation.

## Démarche projet

Le passage du POC au MVP est organisé selon une démarche hybride,
incrémentale et jalonnée.

Il ne s’agit pas d’un cycle en V strict : les fonctionnalités sont
développées et validées progressivement, avec des tests et des revues
réalisés tout au long du projet.

Le planning cible est exprimé à partir d’un instant initial **T0** et couvre
une durée estimée de **12 semaines**.

Les principaux jalons sont :

1. cadrage validé ;
2. architecture et socle technique validés ;
3. socle RAG industrialisé ;
4. personnalisation et mémoire validées ;
5. enrichissement web validé ;
6. version intégrée disponible ;
7. recette du MVP prononcée ;
8. décision Go/No-Go.

## Macro-backlog

Le macro-backlog distingue deux niveaux de priorité.

### Must-Have

Les fonctionnalités indispensables au MVP comprennent notamment :

- API applicative ;
- industrialisation du moteur RAG ;
- stockage persistant des conversations ;
- gestion du contexte géographique ;
- recherche web contrôlée ;
- authentification et sécurité ;
- journalisation des traitements ;
- supervision technique ;
- tests automatisés ;
- déploiement reproductible.

### Nice-to-Have

Les fonctionnalités de confort ou pouvant être reportées comprennent :

- personnalisation avancée des recommandations ;
- interface d’administration complète ;
- tableaux de bord métier enrichis ;
- expérimentation de nouveaux agents ou modèles ;
- mécanismes avancés d’optimisation des coûts ;
- fonctionnalités multilingues.

Cette distinction correspond à une simplification de la méthode MoSCoW :
les Must-Have regroupent les exigences indispensables, tandis que les
Nice-to-Have couvrent principalement les Should-Have et Could-Have.

## Architecture cible

L’architecture proposée repose sur une approche cloud native et modulaire.

Les responsabilités principales sont séparées entre plusieurs composants :

- interface utilisateur ;
- API applicative ;
- moteur RAG ;
- service de mémoire conversationnelle ;
- service de géolocalisation ;
- service de recherche web ;
- stockage relationnel ;
- stockage vectoriel ;
- collecte des logs et métriques ;
- outils de supervision.

Cette séparation permet d’éviter un composant monolithique et facilite
l’évolution indépendante des fonctionnalités.

## Principes d’architecture

### Modularité

Chaque fonction principale est isolée derrière une interface clairement
définie.

Le moteur RAG peut ainsi évoluer sans imposer une refonte de la mémoire,
de la géolocalisation ou de la recherche web.

### Scalabilité

Les services applicatifs doivent rester sans état lorsque cela est
possible.

Les conversations, documents, vecteurs et métadonnées sont stockés dans
des services persistants indépendants, ce qui permet de multiplier les
instances applicatives selon la charge.

### Services managés

Le recours à des services managés réduit les tâches d’administration et
facilite la disponibilité, la sauvegarde et la supervision de
l’infrastructure.

Ce choix doit néanmoins être équilibré avec les contraintes budgétaires
et les risques de dépendance au fournisseur cloud.

### Observabilité

Les logs, métriques et traces doivent être centralisés afin de suivre
l’ensemble d’une requête, depuis l’API jusqu’au modèle de langage et aux
sources externes.

### Sécurité

L’architecture doit intégrer :

- authentification des utilisateurs ;
- gestion des autorisations ;
- chiffrement des échanges ;
- protection des secrets ;
- journalisation des accès ;
- limitation du nombre de requêtes ;
- suppression ou anonymisation des données personnelles ;
- respect des principes du RGPD.

## Flux fonctionnel simplifié

Le fonctionnement général du MVP peut être résumé ainsi :

1. l’utilisateur soumet une requête ;
2. l’API authentifie et valide la demande ;
3. le contexte conversationnel est chargé ;
4. les contraintes géographiques sont déterminées ;
5. le moteur RAG recherche les documents pertinents ;
6. la recherche web complète éventuellement le contexte ;
7. le modèle génère une réponse ;
8. la réponse et les métadonnées sont enregistrées ;
9. les métriques techniques sont envoyées au système de monitoring ;
10. la réponse est retournée à l’utilisateur.

## Gestion des risques

Les principaux risques identifiés concernent :

- l’augmentation des coûts liés aux modèles de langage ;
- les temps de réponse des services externes ;
- les quotas des API ;
- la qualité variable des résultats web ;
- les hallucinations du modèle ;
- la perte ou la fuite de données conversationnelles ;
- la dépendance à un fournisseur cloud ;
- la complexité d’intégration des différents services ;
- la sous-estimation de la charge de développement ;
- la montée en charge non anticipée.

Pour chaque risque, le rapport associe une probabilité, un impact et une
mesure de mitigation.

Les mesures prévues comprennent notamment :

- mise en cache ;
- limitation des appels externes ;
- suivi des consommations ;
- validation des sources ;
- tests automatisés ;
- gestion centralisée des secrets ;
- sauvegarde des données ;
- quotas par utilisateur ;
- alertes budgétaires ;
- déploiement progressif.

## Estimation des coûts

L’étude budgétaire distingue deux catégories.

### Coût de construction — BUILD

Le coût BUILD couvre notamment :

- cadrage et conception ;
- développement de l’API ;
- industrialisation du système RAG ;
- mise en place de la mémoire conversationnelle ;
- intégration de la géolocalisation ;
- intégration de la recherche web ;
- mise en place des tests ;
- configuration de l’infrastructure ;
- observabilité ;
- documentation et recette.

### Coûts d’exploitation — OPEX

Les coûts OPEX couvrent notamment :

- calcul applicatif ;
- stockage relationnel ;
- stockage vectoriel ;
- appels aux modèles de langage ;
- appels aux API externes ;
- stockage des logs ;
- supervision ;
- réseau et transfert de données ;
- sauvegardes.

L’estimation est présentée dans un fichier de type Excel selon plusieurs
hypothèses de charge utilisateurs.

## Optimisation budgétaire

Plusieurs leviers permettent de réduire les coûts du MVP :

- dimensionnement initial limité ;
- montée en charge progressive ;
- extinction des environnements non utilisés ;
- mutualisation des appels au modèle ;
- limitation de la taille du contexte ;
- mise en cache des recherches fréquentes ;
- indexation planifiée plutôt qu’en temps réel lorsque cela est possible ;
- conservation limitée des logs ;
- alertes de dépassement budgétaire ;
- suivi du coût moyen par requête.

## Livrables

Le Projet 13 comprend plusieurs livrables complémentaires :

- rapport de gestion du passage du POC au MVP ;
- analyse des besoins et du contexte ;
- planning du projet sur 12 semaines ;
- macro-backlog des fonctionnalités ;
- registre des risques ;
- architecture technique détaillée ;
- schémas de flux et d’infrastructure ;
- estimation des coûts BUILD et OPEX ;
- bilan des choix techniques et méthodologiques ;
- support de soutenance ;
- portfolio professionnel publié avec Jekyll et GitHub Pages.

## Organisation du dépôt

La première version publiée du dépôt est organisée de la manière suivante :

```text
.
├── livrables/
│   ├── estimation-couts-build-opex.xlsx
│   └── macro-backlog-fonctionnalites.xlsx
├── presentation/
│   └── projet-13-rag-poc-to-mvp-project.pdf
├── rapport/
│   └── rapport-gestion-projet-13.pdf
├── .gitignore
└── README.md
```

## Portfolio professionnel

Le portfolio est développé avec Jekyll et publié depuis un dépôt GitHub
Pages.

Il permet de présenter de manière homogène :

- les projets réalisés ;
- les problématiques métier ;
- les architectures mises en œuvre ;
- les technologies utilisées ;
- les résultats obtenus ;
- les compétences démontrées ;
- les liens vers les dépôts GitHub.

Le code source du portfolio est disponible dans le
[dépôt GitHub](https://github.com/ericginez/ericginez.github.io).

## Compétences démontrées

- collecte et formalisation des besoins métier ;
- analyse des limites d’un POC ;
- définition du périmètre d’un MVP ;
- planification d’un projet Data ;
- élaboration d’un macro-backlog ;
- estimation qualitative de la complexité ;
- gestion des priorités ;
- identification et mitigation des risques ;
- conception d’une architecture cloud scalable ;
- prise en compte de la sécurité et du RGPD ;
- définition d’une stratégie d’observabilité ;
- estimation des coûts BUILD et OPEX ;
- optimisation budgétaire ;
- documentation des choix techniques ;
- communication auprès des parties prenantes ;
- création et publication d’un portfolio professionnel.
