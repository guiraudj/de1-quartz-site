---
title: DE1 — Final Project Report
---

# **DE1 — Final Project Report: Local Lakehouse & Optimization**

Authors: Justine Guirauden and Volcy Desmazures

Course: Data Engineering I \- ESIEE 2025-2026

## **1\. Use‑case and Dataset**

Problem Statement

The objective is to build a scalable data pipeline to analyze the evolution of nutritional quality in global food products. We transform raw, messy, crowdsourced data into structured analytics tables (Gold Layer) to answer business questions such as:

* *"Are products getting sweeter over time?"* (Global Trend Analysis)  
* *"Which country has the fattyest products?"* (Regional Analysis)  
* *"What is the real sugar/fat content for products graded A vs E?"* (Quality Analysis)

**Dataset**

* **Source:** Open Food Facts (World Food Products).  
* **Format:** Raw CSV (Tab-separated values), highly denormalized.  
* **Volume:** \~1.1 GB (Raw uncompressed), containing over **150 columns** per product and millions of rows.  
* **Characteristics:** High level of "messiness": schema drift, mixed types in columns, and significant sparsity (frequent NULL values in nutritional fields).  
* **Known Issues:** Key numerical columns (e.g., fat\_100g) are initially typed as strings; missing "Salt" values despite "Sodium" availability.

## **2\. System and SLOs**

**System Configuration**

* **Engine:** PySpark (Spark 3.x/4.x), Local Single Node mode.  
* **Storage:** Parquet (Snappy compression) for Silver and Gold layers.  
* **Hardware:** Standard Laptop Environment.

**Service Level Objectives (SLOs)**

1. **Storage Efficiency:** Gold Parquet total size must be **\<= 60%** of the Raw CSV baseline.  
2. **Query Performance (Write):** Optimized pipelines should show measurable gains (reduction in Shuffle or Time) on global aggregations (Q3).  
3. **Read Latency:** Achieve sub-second latency (\< 500ms) for filtered queries on Gold tables (Proof of Data Skipping).

## **3\. Lakehouse Design**

**Lineage Diagram**

\[Raw CSV\] \--\> (Bronze) \--\> \[PySpark Cleaning\] \--\> (Silver) \--\> \[Aggregations\] \--\> (Gold Q1/Q2/Q3)

### **Bronze Layer (Landing)**

* **Design:** Immutable ingestion. The raw CSV is copied "as-is" to outputs/project/bronze.  
* **Handling:** Configured to handle strict tab-separation (\\t) and ignore hidden artifacts (:Zone.Identifier) to prevent read errors.

### **Silver Layer (Cleaning & Typing)**

We enforce a strict schema contract to prepare data for aggregation.

| Category | Column | Role | Target Type |
| :---- | :---- | :---- | :---- |
| **Identifier** | code | Primary Key | String |
| **Metrics** | fat\_100g | Metric (Fat content) | Double |
|  | sugars\_100g | Metric (Sugar content) | Double |
| **Dimension** | countries\_en | Partitioning Key | String |
|  | created\_t | Time-series filtering | Date |

* **Data Quality:** Implemented defensive logic for Salt: COALESCE(salt, sodium \* 2.5) to backfill missing values.  
* **Null Policy:** Dropped rows where primary keys (countries\_en) or key metrics are NULL.

### **Gold Layer (Analytics)**

Three datamarts were built to serve specific analytical needs:

* **Q1 (Regional):** Aggregated nutrition by **Country/Year**.  
* **Q2 (Quality):** Aggregated nutrition by **Nutriscore** (Profile A-E).  
* **Q3 (Trends):** Global nutritional trends by **Year**.

## **4\. Physical Design**

We implemented a targeted physical layout strategy to optimize downstream consumption:

1. **Partitioning:** Q1 is partitioned by countries\_en to optimize regional filtering.  
2. **Sorting (sortWithinPartitions):** We applied explicit sorting by **Year** (Q1, Q3) and **Nutriscore** (Q2) before writing. This aligns data on disk to enable **Parquet Data Skipping** (Predicate Pushdown).  
3. **Narrow Projection (Crucial):** We explicitly selected only the required columns (year, sugars\_100g) *before* the Shuffle phase in Q3.

**Evidence of Physical Plan Change:**

The DAG below shows the injection of the Sort operator in the optimized pipeline, absent in the baseline.

*Evidence : Screenshot in metrics/Q1/Optimized/Q1\_optimized\_sort*

## **5\. Evidence and Metrics**

We compared a "Baseline" pipeline (naive read/write) against an "Optimized" pipeline (Projection \+ Sort).

### **Metrics Log Summary**

| Query | Phase | Write Duration (ETL) | Read Latency (User) | Notes |
| :---- | :---- | :---- | :---- | :---- |
| **Q1** | Baseline | \~26.7 s | 1241 ms | High overhead due to small data volume. |
| **Q1** | Optimized | \~29.8 s | **637 ms** | **Write: Slower (+11%)**. **Read: 2x Faster**. Validates trade-off. |
| **Q2** | Baseline | \~28.7 s | 549 ms | Full scan required to find specific Nutriscore. |
| **Q2** | Optimized | \~31.5 s | **272 ms** | **Write: Slower (+10%)**. **Read: 2x Faster**. Data Skipping active. |
| **Q3** | Baseline | **38.2 s** | 784 ms | Full scan required. |
| **Q3** | Optimized | **24.5 s** | **228 ms** | **Write: \-36%**. **Read: 3.4x Faster**. Massive gain from Projection. |

### **Storage Efficiency**

We successfully reduced the storage footprint from **1126 MB (CSV) to 0.34 MB (Parquet)**. This represents a **0.03% compression ratio**, which validates the SLO (\< 60%) by a wide margin.

## **6\. Results and Limits**

### **Storage Optimization Success**

One of the most significant achievements of this pipeline is the extreme storage efficiency. We successfully compressed the dataset from a raw footprint of 1126 MB (CSV) down to a mere 0.34 MB (Gold Parquet). This represents a compression ratio of 0.03%, validating the storage SLO (\<=60%) by a wide margin. This 99.9% reduction is driven by three factors:

1. Aggressive Filtering (Silver): The strict null policy removed sparse rows lacking critical nutritional data.  
2. Column Pruning (Gold): Selecting only the 5 analytical columns needed (out of \>150 raw columns) drastically reduced the data width.  
3. Parquet Encoding: The combination of Snappy compression and dictionary encoding on low-cardinality columns (like countries\_en) proved highly effective compared to verbose text-based CSV.

### **Success: Massive Performance Gains on Q3**

The global trend analysis (Q3) demonstrated the most significant improvement, validating our optimization strategy:

1. **Write Phase (-36% Gain):** By applying **Narrow Projection** early, we reduced the write time from **38s to 24.5s**. This proves that removing unused text columns before the Shuffle phase drastically reduces the I/O throughput required for the aggregation.  
   *Evidence: Q3 baseline vs Q3 optimized → Screenshots in metrics/Q3/Baseline/Q3\_baseline\_schema and in Optimized/Q3\_optimized\_schema*  
2. **Read Phase (3.4x Faster):** The Read Benchmark confirmed that sorting data by year reduced query latency from **784ms to 228ms**. This validates that **Data Skipping** was effectively triggered: Spark read the file metadata and skipped irrelevant row groups instead of scanning the whole file.

Comparison of the physical plans reveals that they are textually identical regarding the `FileScan` and `Filter` operators. This indicates that Spark's **Catalyst Optimizer** successfully pushed down the column pruning in both scenarios. However, the 35% performance gain (38s \-\> 24s) suggests that our explicit optimizations (Projection \+ Sort) improved the **runtime execution efficiency** (likely by reducing memory pressure during the Shuffle phase and optimizing Parquet encoding), distinct from the logical query plan.

### **Deep Dive: The Optimization Paradox (Q1 & Q2)**

For Q1 and Q2, the optimized write time increased slightly (\~10-11%).

* **Analysis:** This is due to the sortWithinPartitions operation which adds CPU overhead during the write phase. However, this is a deliberate **"Write-Once, Read-Many"** physical design choice.  
* **Catalyst Behavior:** Comparison of Physical Plans revealed that Spark's Catalyst Optimizer automatically applied column pruning (Narrow Projection) even in the Baseline query, neutralizing potential I/O gains at write time.  
* **Conclusion:** Although the execution metrics (latency) remain similar between Baseline and Optimized runs for Q1/Q2, the investment pays off downstream. As shown in the benchmarks, sorting the data physically reduces read latency by **50% (2x)** for end-users. Thus, the optimization validates a robust physical design for scalability.

### **Limits: Task Granularity Overhead**

*Evidence : Screenshot in metrics/Q1/Optimized/Q1\_optimized\_output*

The screenshot reveals a job duration of 3s for a negligible output size (\~300KB), confirming that execution time is dominated by Spark's scheduling overhead rather than actual data processing.

Profiling the Spark UI revealed a median task duration of \~6ms with 200 partitions.

* **Analysis:** The metrics show that the default parallelism (spark.sql.shuffle.partitions \= 200\) is too high for the filtered dataset size (\~0.3 MB). Most tasks are effectively empty. This means the job duration is dominated by the **scheduler overhead** (launching tasks) rather than actual data processing.

### **Future Work**

To address the overhead limit, future optimization would involve implementing **Adaptive Query Execution (AQE)** tuning or manual **Coalescing** (.coalesce(1)) for small Gold tables. This would reduce the number of small files (compaction) and improve the task-to-data ratio.

