# 🏦 Sales Transaction Data Warehouse & Refresh (SQL)

> Designing and implementing a normalized data warehouse schema for sales transactions, including dimension tables, fact tables, and an automated incremental refresh pipeline.

---

## 📌 Problem Statement

Businesses need reliable, queryable data warehouses to support business intelligence and reporting. This project designs a star-schema data warehouse for sales transactions and implements an ETL pipeline that supports incremental data refreshes — a core pattern in production data engineering.

---

## 📊 Dataset

| Detail | Info |
|--------|------|
| Domain | Retail Sales Transactions |
| Schema Type | Star Schema (Fact + Dimensions) |
| Tables | `fact_sales`, `dim_customer`, `dim_product`, `dim_date`, `dim_store` |

---

## 🧠 Approach

1. **Schema Design** — Star schema with normalized dimension tables
2. **Data Loading** — Bulk insert scripts for initial load
3. **ETL Pipeline** — Stored procedures for incremental refresh
4. **Query Layer** — Analytical queries for KPIs and reporting

---

## 📈 Key Queries Implemented

- Total revenue by product category and region
- Month-over-month sales growth
- Top 10 customers by lifetime value
- Store performance comparison
- Incremental refresh detecting new/updated records

---

## 🛠️ Tools

**Language:** SQL · **Environment:** MySQL / PostgreSQL · **Concepts:** ETL, Star Schema, Stored Procedures, Indexing

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd "Sales-Data-Warehouse-SQL"
# Execute scripts in order: 01_schema.sql → 02_load.sql → 03_refresh.sql
```

---

## 🔑 Key Takeaways

- Star schema enables fast aggregation queries without complex joins
- Incremental refresh logic prevents full reloads and reduces compute cost
- Proper indexing on foreign keys dramatically improves query performance

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
