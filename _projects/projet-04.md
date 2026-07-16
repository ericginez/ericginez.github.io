---
title: "Auditez un environnement de données"
subtitle: "Identification et correction d’un écart de chiffre d’affaires entre Power BI et un ERP"
project_number: 4
order: 4
status: "Terminé"

summary: >-
  Audit d’un environnement analytique de ventes composé d’extractions OLAP,
  d’une base SQLite, de journaux techniques et d’un tableau de bord Power BI.
  L’analyse a permis d’identifier une erreur de modélisation : les ventes
  historiques étaient valorisées avec le prix courant du produit au lieu du
  prix réellement appliqué lors de la transaction. Le modèle a été corrigé
  afin de réconcilier le chiffre d’affaires Power BI avec celui de l’ERP.

domain:
  - Data Engineering
  - Audit de données
  - Business Intelligence
  - Qualité des données
  - Modélisation relationnelle
  - Sécurité des données

technologies:
  - Power BI
  - DAX
  - SQLite
  - SQL
  - SQL Power Architect
  - Python
  - OpenPyXL
  - PowerShell
  - Git
  - GitHub

github_url: "https://github.com/ericginez/sales-data-environment-audit"
---

## Contexte

L’environnement audité alimente un tableau de bord Power BI à partir de
données de ventes provenant d’une extraction OLAP.

Les données sont organisées autour :

- d’une table de ventes ;
- d’un référentiel produit ;
- d’une dimension client ;
- d’une dimension employé ;
- d’une dimension calendrier ;
- de journaux retraçant les modifications de données.

Un écart avait été constaté entre le chiffre d’affaires présenté dans Power BI
et celui calculé par l’ERP.

L’objectif du projet était d’identifier la cause de cet écart, de contrôler la
cohérence de l’environnement de données et de proposer un modèle corrigé.

## Objectifs

L’audit devait permettre de :

- inventorier les sources disponibles ;
- analyser les volumes et la structure des données ;
- contrôler l’intégrité de la base SQLite ;
- comparer les différentes extractions OLAP ;
- vérifier les clés métier et les relations ;
- identifier la cause racine de l’écart de chiffre d’affaires ;
- corriger la modélisation analytique ;
- documenter le modèle cible ;
- protéger les données sensibles avant publication.

## Environnement analysé

Les fichiers de travail comprenaient :

- deux extractions OLAP au format Excel ;
- un dictionnaire de données ;
- un fichier complet de journaux d’activité ;
- un extrait de journaux ;
- une base SQLite ;
- un modèle relationnel SQL Power Architect ;
- un rapport Power BI.

Les volumes observés étaient les suivants :

| Objet | Nombre de lignes |
|---|---:|
| Ventes | 41 377 |
| Produits | 18 040 |
| Clients | 2 297 |
| Employés | 56 |
| Calendrier | 1 999 |
| Logs | 207 489 |

La base SQLite a passé le contrôle d’intégrité :

```text
PRAGMA integrity_check = ok
```

## Modèle initial

Le modèle initial comportait une table de faits `vente` reliée à quatre
dimensions :

```text
vente.client_id  -> client.id
vente.employe_id -> employe.id
vente.ean        -> produit.ean
vente.date       -> calendrier.date
```

La table de faits contenait :

```text
vente
├── id
├── client_id
├── employe_id
├── ean
├── date
└── ticket_id
```

Le prix était uniquement enregistré dans la dimension produit :

```text
produit.prix
```

Cette colonne représentait le prix courant du catalogue.

## Anomalie identifiée

Le chiffre d’affaires Power BI était calculé à partir du prix courant du
produit.

Lorsqu’un tarif était modifié dans le référentiel produit, les anciennes ventes
étaient implicitement réévaluées avec ce nouveau tarif.

Le calcul revenait conceptuellement à utiliser :

```DAX
Chiffre d'affaires incorrect =
SUMX (
    'Vente Détail',
    RELATED ( Produits[prix] )
)
```

Cette formule ne conservait pas le prix réellement facturé au moment de la
transaction.

La conséquence était un écart entre :

```text
Chiffre d’affaires Power BI
```

et :

```text
Chiffre d’affaires ERP
```

## Comparaison des extractions OLAP

Les deux extractions OLAP ont été comparées feuille par feuille et par clé
métier.

Les contrôles ont montré que :

- les 41 377 ventes étaient identiques sur les colonnes communes ;
- les 18 040 produits étaient identiques ;
- les 2 297 clients étaient identiques ;
- les 56 employés étaient identiques ;
- les 1 999 dates étaient identiques ;
- aucune clé n’était absente ;
- aucune clé n’était dupliquée.

La seconde extraction ajoutait uniquement une colonne `prix` dans la table de
faits `Vente Détail`.

## Analyse du prix de transaction

La colonne ajoutée dans la table de faits a été comparée au prix courant du
référentiel produit.

| Contrôle | Résultat |
|---|---:|
| Ventes contrôlées | 41 377 |
| Produits absents du référentiel | 0 |
| Prix de vente manquants | 0 |
| Prix différents du prix courant | 40 477 |
| Prix identiques au prix courant | 900 |

La majorité des transactions possédait donc un prix différent du prix
actuellement enregistré dans la table produit.

Ce résultat a confirmé que les deux valeurs avaient des significations métier
distinctes :

```text
produit.prix_courant = prix actuel du catalogue
vente.prix_vente     = prix appliqué lors de la transaction
```

## Cause racine

La cause racine était une confusion entre une propriété de dimension et une
mesure transactionnelle.

Le prix courant appartient à la dimension produit. Il décrit l’état actuel du
catalogue.

Le prix de vente appartient à la table de faits. Il doit conserver la valeur
réellement facturée lors de chaque transaction.

L’absence du prix historique dans la table de faits rendait impossible une
restitution fidèle du chiffre d’affaires passé.

## Correction appliquée

La correction a consisté à intégrer le prix de transaction dans la table de
faits.

Le calcul du chiffre d’affaires devient :

```DAX
Chiffre d'affaires =
SUM ( 'Vente Détail'[prix] )
```

Le tableau de bord utilise ainsi le montant réellement associé à chaque vente,
indépendamment des évolutions ultérieures du catalogue.

Cette correction a permis de réconcilier le chiffre d’affaires Power BI avec
celui de l’ERP.

## Modèle cible

Le schéma corrigé distingue explicitement les deux notions de prix :

```text
produit.prix_courant
vente.prix_vente
```

La table de faits cible est structurée ainsi :

```text
vente
├── id
├── client_id
├── employe_id
├── ean
├── date
├── ticket_id
└── prix_vente
```

Elle conserve les quatre relations dimensionnelles :

```text
vente.client_id  -> client.id
vente.employe_id -> employe.id
vente.ean        -> produit.ean
vente.date       -> calendrier.date
```

Le stockage du prix historique dans la table de faits garantit la stabilité du
chiffre d’affaires dans le temps.

## Contrôles complémentaires

### Intégrité relationnelle

La base SQLite contient :

- une clé primaire sur chaque dimension ;
- quatre clés étrangères dans la table `vente` initiale ;
- une table enrichie contenant le prix de transaction.

Le schéma cible rétablit explicitement les quatre clés étrangères et ajoute des
index sur les colonnes utilisées pour les jointures.

### Journaux d’activité

Le fichier complet contient :

```text
207 489 lignes distinctes
```

L’autre fichier de logs contient 5 207 lignes qui sont toutes présentes dans le
fichier complet.

Il a donc été identifié comme un extrait et non comme une source
complémentaire.

### Dictionnaire de données

Le dictionnaire a permis de confirmer :

- les types attendus ;
- les longueurs maximales ;
- les clés primaires ;
- les clés étrangères ;
- les règles de non-nullité ;
- la signification métier des colonnes.

## Sécurité et confidentialité

Les sources analysées contiennent notamment :

- des noms et prénoms ;
- des adresses électroniques ;
- des identifiants utilisateurs ;
- des hashes de mots de passe ;
- des détails de modifications enregistrés dans les logs ;
- des données importées dans le fichier Power BI.

Ces fichiers ne sont pas publiés dans le dépôt GitHub.

Ils sont conservés dans un dossier local exclu par `.gitignore` :

```text
local_data/
```

Le dépôt public contient uniquement :

- le rapport d’audit ;
- le dictionnaire de données public ;
- le modèle relationnel initial ;
- le schéma SQL corrigé ;
- les constats agrégés ;
- les règles métier.

## Organisation du dépôt

```text
.
├── docs/
│   ├── dictionnaire-donnees.md
│   └── rapport-audit.md
├── model/
│   ├── modele_initial.architect
│   └── schema_corrige.sql
├── .gitattributes
├── .gitignore
└── README.md
```

## Livrables

Le projet comprend :

- un modèle SQL Power Architect représentant l’état initial ;
- un schéma SQL documentant le modèle corrigé ;
- un rapport d’audit détaillé ;
- un dictionnaire de données ;
- un tableau de bord Power BI corrigé, conservé localement ;
- une documentation de la cause racine et de la correction ;
- un dépôt public dépourvu de données sensibles.

## Résultats obtenus

L’audit a permis de :

- contrôler 41 377 transactions ;
- valider l’intégrité de la base SQLite ;
- confirmer l’absence de produit orphelin ;
- confirmer l’absence de prix de transaction manquant ;
- comparer les deux versions de l’extraction OLAP ;
- identifier 40 477 transactions dont le prix historique diffère du prix
  courant ;
- isoler la cause de l’écart de chiffre d’affaires ;
- corriger le calcul Power BI ;
- proposer un modèle relationnel cible ;
- sécuriser la publication du projet.

## Recommandations

Les recommandations principales sont :

- conserver systématiquement le prix de transaction dans la table de faits ;
- ne jamais recalculer les ventes historiques avec le prix courant ;
- documenter les règles métier associées aux mesures Power BI ;
- automatiser la réconciliation du chiffre d’affaires avec l’ERP ;
- vérifier régulièrement les clés orphelines ;
- ajouter des index sur les clés étrangères ;
- appliquer un contrôle d’accès strict aux logs ;
- exclure les données personnelles et les hashes de mots de passe des dépôts
  publics.

## Limites

Le projet présente plusieurs limites :

- les contrôles ont été réalisés sur des extractions statiques ;
- la réconciliation avec l’ERP n’est pas automatisée ;
- aucun pipeline d’audit continu n’a été développé ;
- les données sources ne peuvent pas être fournies publiquement ;
- le modèle Power BI corrigé est conservé localement en raison des données
  embarquées ;
- la table des employés contient des attributs qui devraient être mieux isolés
  et protégés ;
- la gestion de l’historique des autres attributs produit n’a pas été étudiée.

## Évolutions possibles

Les évolutions envisagées comprennent :

- automatiser les contrôles avec Python ou SQL ;
- produire un rapport d’audit à chaque nouvelle extraction ;
- comparer automatiquement les indicateurs avec ceux de l’ERP ;
- intégrer des tests de non-régression sur le chiffre d’affaires ;
- ajouter des contrôles d’intégrité référentielle ;
- historiser les autres attributs susceptibles d’évoluer ;
- créer une couche de données dédiée à la BI ;
- pseudonymiser les données personnelles ;
- définir une politique de rétention des logs ;
- mettre en place une supervision de la qualité des données.

## Compétences développées

Ce projet met en œuvre les compétences suivantes :

- audit d’un environnement de données ;
- analyse exploratoire de sources hétérogènes ;
- lecture et contrôle de classeurs Excel ;
- inspection d’une base SQLite ;
- comparaison d’extractions OLAP ;
- analyse d’un modèle relationnel ;
- contrôle des clés métier ;
- diagnostic d’un indicateur BI ;
- identification d’une cause racine ;
- modélisation d’une table de faits ;
- historisation d’une mesure transactionnelle ;
- développement de mesures DAX ;
- rédaction d’un dictionnaire de données ;
- documentation technique ;
- gestion des données sensibles ;
- sécurisation d’un dépôt GitHub.