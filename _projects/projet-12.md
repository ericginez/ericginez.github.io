---
title: "Gérez un projet d’infrastructure"
subtitle: "Pipeline ELT automatisé pour le suivi d’activités sportives"
project_number: 12
order: 12
status: "Terminé"

summary: >-
  Conception et automatisation d’une infrastructure de données
  complète reposant sur une architecture Bronze–Silver–Gold,
  orchestrée avec Kestra et restituée dans Power BI.

domain:
  - Data Engineering
  - Orchestration
  - Business Intelligence
  - Qualité des données

technologies:
  - Python
  - PostgreSQL
  - Docker
  - Kestra
  - Power BI
  - Pytest
  - GitHub

github_url: "https://github.com/ericginez/sports-benefits-data-pipeline"
---

## Contexte

Une entreprise souhaite encourager la pratique sportive de ses employés
et automatiser l’attribution d’avantages liés à leur activité physique.

La solution doit intégrer les données des ressources humaines, simuler
des activités sportives, calculer les distances domicile-travail, produire
des notifications et fournir des indicateurs consolidés aux équipes métier.

## Objectifs

Le projet répond à plusieurs objectifs :

- intégrer des sources de données hétérogènes ;
- automatiser les traitements récurrents ;
- assurer la qualité et la traçabilité des données ;
- produire des tables directement exploitables par Power BI ;
- superviser l’exécution des différents pipelines ;
- gérer les traitements initiaux et incrémentaux.

## Architecture

L’infrastructure repose sur une architecture en trois couches :

### Bronze

La couche Bronze conserve les données chargées ou générées dans leur
forme initiale :

- données des employés ;
- profils sportifs ;
- activités sportives ;
- messages de notification.

### Silver

La couche Silver contient les données nettoyées, validées et enrichies :

- calcul des distances domicile-travail ;
- contrôle des types d’activités ;
- suivi des lots incrémentaux ;
- statut de livraison des notifications.

### Gold

La couche Gold fournit les données métier destinées à la restitution :

- éligibilité des employés ;
- jours de bien-être attribués ;
- montant des primes sportives ;
- agrégats mensuels ;
- indicateurs par sport et unité métier.

## Pipeline automatisé

Deux workflows Kestra ont été mis en place.

Le premier initialise l’environnement en chargeant les données RH,
en calculant les distances, en générant l’historique sportif et en
construisant les tables Gold.

Le second traite les nouveaux lots incrémentaux, prépare les messages,
envoie les notifications en attente et actualise les indicateurs métier.

## Qualité des données

La solution comprend une suite de tests automatisés couvrant notamment :

- la validité des données RH ;
- la cohérence des modes de transport ;
- les distances domicile-travail ;
- les activités initiales et incrémentales ;
- la préparation et la livraison des messages ;
- la réconciliation des couches Silver et Gold.

Au total, **59 tests automatisés** valident le pipeline complet.

## Résultats

La version finale traite notamment :

- **161 employés** ;
- **3 847 activités historiques** ;
- **83 employés éligibles** ;
- **415 jours de bien-être attribués** ;
- **15 sports suivis** ;
- **574 agrégats mensuels**.

Le reporting Power BI utilise uniquement les tables de la couche Gold,
afin de séparer clairement les traitements techniques de l’exploitation
métier.

## Compétences démontrées

- conception d’une architecture ELT ;
- modélisation des couches Bronze, Silver et Gold ;
- développement de traitements Python ;
- orchestration avec Kestra ;
- gestion de traitements incrémentaux ;
- mise en place de tests automatisés ;
- suivi de la qualité et de la traçabilité ;
- création d’un reporting Power BI ;
- gestion de versions avec Git et GitHub.