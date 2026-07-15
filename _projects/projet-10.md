---
title: "Mettez en place un pipeline d’orchestration des flux"
subtitle: "Automatisation d’un pipeline de données e-commerce avec Kestra et DuckDB"
project_number: 10
order: 10
status: "Terminé"

summary: >-
  Conception d’un pipeline orchestré avec Kestra pour nettoyer,
  rapprocher et analyser des données ERP et e-commerce, contrôler
  leur qualité et produire automatiquement des exports exploitables.

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
  - Docker
  - Git
  - GitHub

github_url: "https://github.com/ericginez/wine-orchestration-pipeline"
---

## Contexte

Une entreprise spécialisée dans la vente de vins dispose de deux sources
de données distinctes :

- un système ERP contenant les références produits, les prix et les
  informations de stock ;
- une plateforme e-commerce contenant les ventes, les identifiants web
  et les informations associées aux produits commercialisés.

Ces sources ne sont pas directement alignées. Les données doivent être
nettoyées, dédupliquées et rapprochées avant de pouvoir calculer des
indicateurs fiables.

L’objectif est d’automatiser cette chaîne de traitement dans un workflow
reproductible et contrôlé.

## Objectifs

Le projet répond aux objectifs suivants :

- charger les données issues de plusieurs sources ;
- contrôler leur structure et leur qualité ;
- nettoyer et normaliser les informations produits ;
- supprimer les doublons ;
- rapprocher les références ERP et web ;
- calculer les indicateurs commerciaux ;
- détecter les valeurs de prix atypiques ;
- produire automatiquement des fichiers de sortie ;
- orchestrer les différentes étapes du pipeline ;
- interrompre l’exécution lorsqu’un contrôle critique échoue.

## Sources de données

Le pipeline traite deux ensembles principaux.

### Données ERP

La source ERP contient notamment :

- les références internes ;
- les prix ;
- les quantités en stock ;
- les informations descriptives des produits.

Après nettoyage et déduplication, la source ERP comprend
**825 références uniques**.

### Données web

La source e-commerce contient notamment :

- les identifiants utilisés sur le site ;
- les références permettant le rapprochement avec l’ERP ;
- les quantités vendues ;
- les informations de commercialisation.

Après nettoyage et déduplication, cette source comprend
**714 références uniques**.

## Architecture du pipeline

Le traitement est orchestré dans Kestra et s’appuie principalement sur
DuckDB pour exécuter les transformations SQL.

Le workflow suit une succession d’étapes clairement séparées :

1. chargement des sources ;
2. contrôle de leur disponibilité ;
3. nettoyage des données ERP ;
4. nettoyage des données web ;
5. déduplication des références ;
6. rapprochement des deux sources ;
7. calcul des indicateurs métier ;
8. détection des prix atypiques ;
9. validation des résultats ;
10. génération des exports.

Cette organisation facilite la compréhension du pipeline, son
débogage et l’identification précise d’une étape défaillante.

## Nettoyage des données

Les traitements de préparation portent notamment sur :

- la normalisation des noms de colonnes ;
- la conversion des types ;
- le contrôle des identifiants ;
- le traitement des valeurs manquantes ;
- la suppression des doublons ;
- la vérification des prix ;
- le contrôle des quantités vendues ;
- l’harmonisation des clés de rapprochement.

Ces opérations permettent d’éviter que des anomalies présentes dans les
sources soient propagées dans les résultats finaux.

## Rapprochement ERP et e-commerce

Le rapprochement permet d’associer les caractéristiques présentes dans
l’ERP aux résultats commerciaux provenant du site web.

Après fusion, le jeu de données consolidé comprend
**714 produits rapprochés**.

Cette table permet notamment de relier :

- le prix unitaire ;
- la quantité vendue ;
- le chiffre d’affaires ;
- le stock disponible ;
- les informations descriptives du produit.

## Calcul des indicateurs

Le chiffre d’affaires par produit est calculé à partir du prix unitaire
et du nombre de ventes :

```text
Chiffre d’affaires = prix × quantité vendue