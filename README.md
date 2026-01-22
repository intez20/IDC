# Project Overview

This project implements a production-style **Medallion Architecture (Bronze → Silver → Gold)** using **Apache Spark** and **Delta Lake** on **Databricks** to process large-scale e-commerce behavioral event data.

The goal is to demonstrate real-world data engineering practices, including incremental processing, data quality enforcement, schema contracts, and analytics-ready data marts.

The source data represents user behavior (views, carts, purchases) from a multi-category online store.

---

## Dataset

- **Source:** Kaggle – *E-commerce Behavior Data from a Multi-Category Store*
- **Data type:** Clickstream / behavioral events
- **Time range used:** Monthly CSV files (e.g., Oct–Nov 2019)
- **Event grain:** One row per user action
- **Dataset URL:**  https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store

---

## High-Level Architecture

```text
       Raw CSV Files
           |
           v
    +------------------+
    |      Bronze      |
    | Raw Delta Tables |
    |  - Immutable     |
    |  - Append-only   |
    |  - Ingestion meta|
    +------------------+
             |
             v
+---------------------------+
|           Silver          |
| Cleaned & Conformed Data  |
|  - Data quality rules     |
|  - Standardization        |
|  - Deduplication          |
|  - Incremental MERGE      |
|                           |
|  +---------------------+  |
|  |  Silver Reject      |  |
|  |  - Invalid records  |  |
|  |  - Rejection reasons|  |
|  +---------------------+  |
+---------------------------+
          |
          v
+---------------------------+
|            Gold           |
| Business Data Marts       |
|  - Revenue KPIs           |
|  - Funnels                |
|  - Product metrics        |
|  - Growth metrics         |
+---------------------------+
```

---

## 🥉 Bronze Layer — Raw Ingestion

### Purpose

The Bronze layer acts as the **system of record**. It stores raw data exactly as received, enabling auditability and reprocessing.

### Key Characteristics

- Delta Lake format  
- Append-only writes  
- No filtering or cleansing  
- Original schema preserved  
- Ingestion metadata added:
  - `ingestion_time`
  - `source_file`
  - `event_date` (derived from `event_time`)

### Why This Matters

- Prevents early data loss  
- Supports replay if downstream logic changes  
- Mirrors production-grade ingestion pipelines  

---

## 🥈 Silver Layer — Cleaned & Conformed Data

### Purpose

The Silver layer transforms raw data into a **trusted, query-ready dataset** by enforcing data quality and consistency rules.

### Transformations Applied

- **Schema enforcement**
- **Standardization**
  - `event_type`, `brand`, `category_code` → lowercase, trimmed
- **Data quality validation**
  - Valid timestamps
  - Non-negative prices
  - Valid event types
  - Required user identifiers
- **Deduplication** using business keys:
  - `(user_session, product_id, event_time, event_type)`

### Incremental Processing

- Event-time watermarking  
- Reprocesses a rolling window to handle late-arriving data  
- Uses Delta `MERGE` for idempotent writes  

---

## 🚧 Silver Reject / Quarantine Table

### Purpose

Records that fail Silver-layer validation rules are written to a reject table instead of being silently dropped.

### Why This Exists

- Full transparency into data quality  
- Debugging and root-cause analysis  
- Prevents bad data from contaminating analytics  

### Reject Metadata Includes

- `reject_rule_id`
- `reject_reason`
- `reject_time`
- `reject_layer`

---

## 🥇 Gold Layer — Business Data Marts

### Purpose

The Gold layer provides **analytics-ready tables** aligned with specific business questions. These tables are designed for BI tools, dashboards, and reporting.

### Gold Tables

| Table Name                  | Description                                   |
|-----------------------------|-----------------------------------------------|
| `gold_daily_revenue`        | Daily revenue and order counts                |
| `gold_daily_active_users`   | Daily active user metrics                     |
| `gold_product_metrics`      | Views, carts, purchases, conversion per item |
| `gold_brand_revenue`        | Brand-level revenue leaderboard               |
| `gold_user_funnel`          | User-level funnel metrics                     |

### Incremental Strategy

- Derived from Silver using `event_date`  
- Date-partitioned tables overwrite only affected partitions  
- Snapshot-style tables fully recomputed for simplicity  
- Uses `saveAsTable()` to preserve Unity Catalog metadata  

---

## 🛠 Technology Stack

- Apache Spark (PySpark)
- Delta Lake
- Databricks
- Unity Catalog
- SQL / Delta MERGE

---

## ✅ Key Engineering Concepts Demonstrated

- Medallion architecture
- Incremental processing with watermarks
- Schema contracts and partitioning discipline
- Data quality enforcement with reject handling
- Idempotent pipelines
- Unity Catalog–compliant table management
- Analytics-focused data modeling
