# 🚀 NovaCart Data Engineering Pipeline (Bronze → Silver → Gold)

## 📌 Project Overview

This project demonstrates a **modern data engineering pipeline** built using **Azure + Databricks + Delta Lake**, implementing the **Medallion Architecture (Bronze, Silver, Gold layers)**.

The pipeline ingests data from a source system, processes it incrementally, ensures data quality, and delivers analytics-ready datasets.

---

## 🏗️ Architecture

```
Source (SQL Server)
        ↓
   Bronze Layer (Raw Ingestion)
        ↓
   Silver Layer (Cleaned + Validated)
        ↓
   Gold Layer (Business Aggregations + SCD2 + Snapshots)
```

---

## ⚙️ Technologies Used

* Apache Spark (PySpark)
* Delta Lake (MERGE, ACID transactions)
* Azure Databricks
* Azure Data Lake / Volumes
* SQL
* Python

---

## 🥉 Bronze Layer (Raw Ingestion)

### ✅ Key Features

* Incremental ingestion using:

  * **Timestamp (`ts_col`)**
  * **Primary Key (`pk_col`)**
* Watermark-based loading
* Metadata tracking via `ingestion_control` table
* Handles:

  * Late arriving data
  * Duplicate timestamps (using PK fallback)

### 📂 Tables

* `orders_raw`
* `products_raw`
* `payments_raw`
* `ingestion_control`

### 🔁 Logic

* Load only new/updated records:

```sql
(ts > last_ts) OR (ts = last_ts AND pk > last_pk)
```

---

## 🥈 Silver Layer (Data Cleaning & Validation)

### ✅ Key Features

* Incremental processing using `bronze_ingested_at`
* Deduplication using window functions
* Data standardization (uppercase, trimming, casting)
* Data validation rules
* Separation into:

  * ✅ Valid data (`*_transformed`)
  * ❌ Invalid data (`*_quarantine`)

### 📂 Tables

* `orders_cleaned`, `orders_transformed`, `orders_quarantine`
* `products_cleaned`, `products_transformed`, `products_quarantine`
* `payments_cleaned`, `payments_transformed`, `payments_quarantine`
* `processing_control`

### 🔍 Example Validations

* Null checks (customer_id, product_id, etc.)
* Invalid amounts (≤ 0)
* Incorrect formats (price, status fields)

---

## 🥇 Gold Layer (Business Logic & Analytics)

### ✅ Key Features

* Incremental processing based on Silver changes
* Impact-based recomputation (only affected records)
* Multi-table joins (Orders + Products + Payments)
* Business metrics:

  * Payment completion ratio
  * Payment state classification

### 📂 Core Tables

* `orders_information`
* `orders_information_scd2`
* `category_performance`
* `processing_control`

---

## 🔄 Slowly Changing Dimension (SCD Type 2)

### ✅ Implementation

* Tracks historical changes in `orders_information_scd2`
* Maintains:

  * `valid_from_ts`
  * `valid_to_ts`
  * `is_current`

### 🔁 Logic

* When record changes:

  1. Expire old record (`is_current = false`)
  2. Insert new version

---

## 📊 Business Metrics

### 🧮 Derived Columns

* `payment_completion_ratio`
* `payment_state`:

  * Unpaid
  * Partially_paid
  * Overpaid
  * Invalid_order_amount

### 📈 Aggregations

* Category-level performance:

  * Total orders
  * GMV (Gross Merchandise Value)
  * Total paid amount
  * Avg payment completion ratio
  * Payment failure rate

---

## 📦 Data Snapshots

### ✅ Features

* Latest snapshot (overwrite)
* Historical snapshots (partitioned by date & timestamp)

### 📂 Storage Structure

```
/gold_snapshots/
    ├── gold_latest/
    └── gold_snapshots/
         └── run_date=YYYY-MM-DD/
              └── run_ts=YYYY-MM-DD HH:MM:SS/
```

---

## 🔁 Incremental Processing Strategy

| Layer  | Strategy                      |
| ------ | ----------------------------- |
| Bronze | Timestamp + PK watermark      |
| Silver | `bronze_ingested_at`          |
| Gold   | `updated_at` / `processed_at` |

---

## 🧠 Key Design Concepts

* Idempotent pipelines
* Metadata-driven processing
* Delta Lake MERGE operations
* Data quality enforcement
* Change Data Capture (CDC-like behavior)
* Scalable incremental architecture

---

## 📌 How to Run

1. Run **Bronze Notebook**
2. Run **Silver Notebook**
3. Run **Gold Notebook**

Ensure:

* Catalog and schemas exist
* Source tables are accessible

---

## 🎯 Outcomes

* Fully automated incremental pipeline
* Clean, validated, analytics-ready data
* Historical tracking with SCD2
* Efficient recomputation using impact analysis

---

## 🔮 Future Improvements

* Add orchestration (Azure Data Factory / Workflows)
* Implement streaming ingestion
* Add data quality monitoring dashboards
* Integrate with Power BI for reporting

---

## 👨‍💻 Author

**Malindu Dilmin**
Applied Sciences Graduate | Data Engineering Enthusiast

---

