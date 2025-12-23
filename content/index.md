---
title: Home
publish: true
---
# Welcome to our Data Engineering Website

> [!abstract] Information
> * **Project Team:** Justine Guirauden and Volcy Desmazures
> * **School:** ESIEE Paris (2025-2026)
> * **Course:** Data Engineering 1 (DE1)

This site documents our practical labs as well as our final project on Big Data pipeline optimization.

---

## Final Project: Local Lakehouse & Optimization

To validate this semester, we built a **local Lakehouse** capable of processing real and complex data while meeting strict performance objectives (SLOs).

### The Topic: Nutritional Analysis (Open Food Facts)
We analyzed the evolution of the nutritional quality of global food products (Sugar, Fat, Nutriscore).
* **Data:** ~1.1 GB of raw CSV, highly denormalized (>150 columns).
* **Stack:** PySpark (Spark 3.x), Parquet, Local Single Node.

### Key Results
We compared a "naive" pipeline (Baseline) against our optimized pipeline (Silver/Gold layers).

| Metric | Result Obtained | Technical Impact |
| :--- | :--- | :--- |
| **Storage** | **-99.9%** (1.1GB -> 0.34MB) | Snappy Compression + Drastic Cleaning |
| **Speed (Q3)** | **x3.4 faster** | *Predicate Pushdown* & *Data Skipping* |
| **Latency** | **228 ms** | Optimized reading via sorting (*sortWithinPartitions*) |

> [!success] Access the Report
> This project demonstrates how rigorous physical design (Sorting, Partitioning, Projection) can transform an unusable dataset into a high-performance Datamart.
>
> [[project-final/DE1 — Final Project Report|Read the Full Project Report]]
>
> [[static/nb/project-final/DE1_Project_Notebook_EN|View the Jupyter Notebook (Source Code)]]

---

## Laboratories (Labs)

Here are all the practical labs completed, covering Data Engineering fundamentals, from containerization to data pipelines.

* **Lab 1: Environment & Docker**
    * *Skills Acquired:* Environment setup, containerization.
    * [[static/nb/labs-final/lab1_assignment/assignment1_esiee.html|Access Lab 1]]

* **Lab 2: SQL & Data Modeling**
    * *Skills Acquired:* Analytical queries, data structuring.
    * [[static/nb/labs-final/lab2_assignment/assignment2_esiee|Access Lab 2]]

* **Lab 3: Data Pipelines**
    * *Skills Acquired:* Orchestration and transformation.
    * [[static/nb/labs-final/lab3_assignment/assignment3_esiee|Access Lab 3]]

---

## About this site

This portfolio is built using the **"Docs as Code"** approach:
* Generated with [Quartz](https://quartz.jzhao.xyz/).
* Hosted on **Cloudflare Pages**.
* Secured by **Cloudflare Zero Trust** (Access Policies).