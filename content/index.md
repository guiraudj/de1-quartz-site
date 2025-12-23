---
title: Home
publish: true
---
---
title: Portfolio Data Engineering - Justine Guirauden
description: Portfolio des projets et laboratoires de Data Engineering 1 (ESIEE Paris).
---

# Bienvenue sur mon Portfolio

> [!abstract] Informations
> * **Binome Projet :** Justine Guirauden et Volcy Desmazures
> * **Ecole :** ESIEE Paris (2025-2026)
> * **Cours :** Data Engineering 1 (DE1)

Ce site documente nos laboratoires pratiques ainsi que notre projet final sur l'optimisation de pipelines Big Data.

---

## Projet Final : Local Lakehouse & Optimization

Pour valider ce semestre, nous avons construit un **Lakehouse local** capable de traiter des données réelles et complexes tout en respectant des objectifs de performance stricts (SLOs).

### Le Sujet : Analyse Nutritionnelle (Open Food Facts)
Nous avons analysé l'évolution de la qualité nutritionnelle des produits alimentaires mondiaux (Sucre, Gras, Nutriscore).
* **Données :** ~1.1 GB de CSV brut (Raw), très dénormalisé (>150 colonnes).
* **Stack :** PySpark (Spark 3.x), Parquet, Local Single Node.

### Résultats Clés
Nous avons comparé un pipeline "naïf" (Baseline) contre notre pipeline optimisé (Silver/Gold layers).

| Metrique | Resultat obtenu | Impact Technique |
| :--- | :--- | :--- |
| **Stockage** | **-99.9%** (1.1GB -> 0.34MB) | Compression Snappy + Nettoyage drastique |
| **Vitesse (Q3)** | **x3.4 plus rapide** | *Predicate Pushdown* & *Data Skipping* |
| **Latence** | **228 ms** | Lecture optimisée via tri (*sortWithinPartitions*) |

> [!success] Accès au rapport
> Ce projet démontre comment une conception physique rigoureuse (Tri, Partitionnement, Projection) peut transformer un jeu de données inutilisable en Datamart performant.
>
> [[project-final/DE1 — Final Project Report|Lire le Rapport Complet du Projet]]
>
> [[project-final/DE1_Project_Notebook_EN|Voir le Notebook Jupyter (Code Source)]]

---

## Laboratoires (Labs)

Voici l'ensemble des travaux pratiques réalisés, couvrant les fondamentaux du Data Engineering, de la conteneurisation aux pipelines de données.

* **Lab 1 : Environnement & Docker**
    * *Acquis :* Installation de l'environnement, conteneurisation.
    * [[labs-final/lab1/assignment1_esiee|Accéder au Lab 1]]

* **Lab 2 : SQL & Modélisation de données**
    * *Acquis :* Requêtes analytiques, structuration de la donnée.
    * [[labs-final/lab2/assignment2_esiee|Accéder au Lab 2]]

* **Lab 3 : Pipelines de Données**
    * *Acquis :* Orchestration et transformation.
    * [[labs-final/lab3/assignment3_esiee|Accéder au Lab 3]]

---

## À propos de ce site

Ce portfolio est construit selon l'approche **"Docs as Code"** :
* Generated with [Quartz](https://quartz.jzhao.xyz/).
* Hosted on **Cloudflare Pages**.
* Secured by **Cloudflare Zero Trust** (Access Policies).