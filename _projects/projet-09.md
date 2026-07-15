---
title: "Concevez un pipeline de données en streaming"
subtitle: "Traitement temps réel de tickets clients avec Redpanda et Apache Spark"
project_number: 9
order: 9
status: "Terminé"

summary: >-
  Conception d’un prototype de pipeline streaming conteneurisé,
  reposant sur Redpanda pour la diffusion des événements et
  Apache Spark Structured Streaming pour leur consommation,
  leur transformation et leur stockage aux formats JSON et Parquet.

domain:
  - Data Engineering
  - Streaming de données
  - Architecture distribuée
  - Traitement temps réel
  - Conteneurisation

technologies:
  - Python
  - Apache Spark
  - Redpanda
  - Kafka API
  - Docker
  - Docker Compose
  - JSON
  - Parquet
---

## Contexte

Une organisation souhaite traiter en continu les tickets transmis par
ses clients afin de disposer rapidement de données exploitables par les
équipes opérationnelles et analytiques.

Une architecture batch classique imposerait d’attendre l’exécution
planifiée d’un traitement avant de rendre les nouveaux événements
disponibles.

Le projet consiste donc à réaliser un prototype de pipeline de données
en streaming capable de :

- produire des événements représentant des tickets clients ;
- transporter ces événements dans un système de messagerie ;
- les consommer en continu ;
- valider et transformer les données reçues ;
- enregistrer les résultats dans des formats adaptés à différents usages.

## Objectifs

Le projet poursuit les objectifs suivants :

- concevoir une architecture événementielle ;
- produire des messages structurés en continu ;
- utiliser un broker compatible avec l’écosystème Kafka ;
- consommer les événements avec Spark Structured Streaming ;
- contrôler la structure et la qualité des messages ;
- transformer les données au fil de leur arrivée ;
- gérer les données incorrectes ou incomplètes ;
- produire des sorties aux formats JSON et Parquet ;
- garantir la reprise des traitements ;
- conteneuriser les différents composants ;
- documenter et valider le fonctionnement du prototype.

## Architecture générale

Le pipeline repose sur trois fonctions principales :

1. un producteur génère ou transmet des tickets clients ;
2. Redpanda reçoit les messages dans un topic ;
3. Spark Structured Streaming consomme, transforme et stocke les données.

Le flux peut être représenté ainsi :

```text
Producteur Python
       │
       ▼
Topic Redpanda
client_tickets
       │
       ▼
Spark Structured Streaming
       │
       ├── validation du schéma
       ├── transformation des données
       ├── gestion des événements invalides
       └── suivi des offsets
       │
       ├── sortie JSON
       └── sortie Parquet