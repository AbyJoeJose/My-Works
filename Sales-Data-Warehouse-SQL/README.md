# 🏦 Sales Transaction Data Warehouse & ETL Pipeline (SQL)

> Designing and implementing a production-style star-schema data warehouse for a retail business handling both **sales and rental transactions** — including dimension tables, a revenue fact table, intermediate ETL staging, aggregate tables, and a daily store performance snapshot — all built and loaded using MySQL.

---

## 📌 Problem Statement

A retail company sells and rents products across multiple stores in different regions. To support business intelligence and reporting, this project designs a complete **data warehouse** from an operational database (`josea_zagimore`) into a clean analytical warehouse (`josea_zagimore_dw`).

The warehouse supports queries on:
- Revenue by product, store, customer, and date
- Sales vs rental transaction analysis
- Daily store performance monitoring
- Footwear category revenue tracking
- High-value transaction counts
- Local vs non-local customer revenue

---

## 🏗️ Architecture Overview

```
Operational DB (josea_zagimore)
        ↓
  Data Staging Layer
  (Dimension tables + IntermediateFact)
        ↓
  Revenue Fact Table
        ↓
  Aggregate & Snapshot Tables
        ↓
  Data Warehouse (josea_zagimore_dw)
```

---

## 📊 Schema Design

### Dimension Tables

| Table | Key Columns | Source Tables |
|-------|-------------|--------------|
| `Customer_Dimension` | CustomerKey (PK) · CustomerID · CustomerName · CustomerZip | `customer` |
| `Store_Dimension` | StoreKey (PK) · StoreID · StoreZip · RegionID · RegionName | `store` + `region` |
| `Product_Dimension` | ProductKey (PK) · ProductID · ProductName · ProductType · VendorID · VendorName · CategoryID · CategoryName · ProductSalesPrice · ProductDailyRentalPrice · ProductWeeklyRentalPrice | `product` + `rentalProducts` + `vendor` + `category` |
| `Calendar_Dimension` | CalendarKey (PK) · FullDate · MonthYear · Year | Pre-populated |

**Note:** `Product_Dimension` is loaded twice — once for `ProductType = 'Sales'` and once for `ProductType = 'Rental'` — allowing the same dimension table to serve both transaction types.

---

### Fact Table

**`Revenue`** — Central fact table storing all transactions

| Column | Description |
|--------|-------------|
| `RevenueKey` | Primary key |
| `UnitsSold` | Number of units (0 for rentals) |
| `RevenueGenerated` | Total revenue for the transaction |
| `RevenueType` | `'Sales'` · `'RentalWeekly'` · `'RentalDaily'` |
| `TID` | Original transaction ID |
| `CustomerKey` | FK → Customer_Dimension |
| `StoreKey` | FK → Store_Dimension |
| `ProductKey` | FK → Product_Dimension |
| `CalendarKey` | FK → Calendar_Dimension |

---

## 🧠 ETL Pipeline

### Step 1 — Populate Dimension Tables

```
customer → Customer_Dimension
store + region → Store_Dimension
product + vendor + category → Product_Dimension (Sales)
rentalProducts + vendor + category → Product_Dimension (Rental)
```

### Step 2 — Intermediate Fact (Staging)

An `IntermediateFact` table is created as a staging buffer before loading into the Revenue fact table. This pattern decouples extraction from dimension key resolution.

**Rental Transactions:**
- **Weekly rentals:** `RevenueGenerated = productpriceweekly × duration` from `rentvia` where `rentaltype = 'W'`
- **Daily rentals:** `RevenueGenerated = productpriceweekly × duration` from `rentvia` where `rentaltype = 'D'`

**Sales Transactions:**
- `RevenueGenerated = productprice × noofitems` from `soldvia` joined with `salestransaction`
- `UnitsSold = noofitems`

### Step 3 — Load Revenue Fact

Joins `IntermediateFact` with all four dimension tables on natural keys (CustomerID, StoreID, ProductID, FullDate) to resolve surrogate keys, then inserts into `Revenue`.

Separate inserts for:
- `ProductType = 'Rental'` (from rental intermediate fact)
- `ProductType = 'Sales'` (from sales intermediate fact)

### Step 4 — Load Warehouse

All staging tables copied to `josea_zagimore_dw`:
- `Customer_Dimension`, `Product_Dimension`, `Store_Dimension`, `Calendar_Dimension`, `Revenue`

---

## 📦 Aggregate & Snapshot Tables

### One-Way Revenue Aggregate by Product Category

Aggregates total units sold and revenue at the **product category level** — pre-computing a commonly needed analytical grouping.

**`product_cat_dimension`** — extracted distinct categories from Product_Dimension with auto-increment key.

**`OneWayRevenueAggregateByProductCategory`**

| Column | Description |
|--------|-------------|
| `TotalUnitSold` | SUM of units |
| `TotalRevenueGenerated` | SUM of revenue |
| `CalendarKey` | FK |
| `CustomerKey` | FK |
| `StoreKey` | FK |
| `Product_Cat_Key` | FK → product_cat_dimension |

**Composite PK:** (CalendarKey, CustomerKey, StoreKey, Product_Cat_Key)

---

### Daily Store Snapshot

A **periodic snapshot fact table** capturing daily store-level KPIs — built incrementally by adding columns via `ALTER TABLE` and updating from temporary helper tables.

**`DailyStoreSnapshot`**

| Column | How Computed |
|--------|-------------|
| `TotalUnitSold` | SUM of UnitsSold per store per day |
| `TotalRevenueGenerated` | SUM of RevenueGenerated per store per day |
| `TotalNumberOfTransaction` | COUNT(DISTINCT TID) per store per day |
| `AverageRevenueGenerated` | AVG of RevenueGenerated (DECIMAL 9,2) |
| `TotalFootwearRevenue` | Revenue filtered on `CategoryName = 'Footwear'` via temp table |
| `NumberOfHVTransaction` | COUNT(DISTINCT TID) where `RevenueGenerated > 100` |
| `TotalLocalRevenue` | Revenue where first 2 digits of StoreZip = first 2 digits of CustomerZip |

**Composite PK:** (CalendarKey, StoreKey)

**Build pattern used:**
1. Create base snapshot with core aggregates
2. Create temporary helper tables (`FootwearRevenue`, `HVTransaction`, `TotalLocalRevenue`)
3. `ALTER TABLE` to add new columns with `DEFAULT 0`
4. `UPDATE ... SET` using joined helper tables
5. `DROP TABLE` to clean up helpers
6. Copy final snapshot to DW

---

## 🔑 Key SQL Techniques Used

| Technique | Where Used |
|-----------|-----------|
| Star schema design | 4 dimensions + 1 fact table |
| Surrogate keys (AUTO_INCREMENT) | All dimension and aggregate tables |
| Intermediate/staging fact table | Decouples extraction from key resolution |
| `INSERT INTO ... SELECT` | All ETL loading steps |
| Multi-table `FROM` with `WHERE` joins | Dimension population (pre-ANSI join style) |
| `ProductType` flag | Single Product_Dimension serves both Sales and Rental |
| `LEFT(zip, 2)` comparison | Local customer detection in snapshot |
| `ALTER TABLE ADD COLUMN` | Incremental snapshot column addition |
| `UPDATE ... SET` with multi-table join | Snapshot column updates from helpers |
| `DROP TABLE` | Cleanup of temporary helper tables |
| `ADD FOREIGN KEY` | Referential integrity in DW |
| `ADD PRIMARY KEY` | Composite PKs on aggregate and snapshot tables |
| `DECIMAL(9,2)` column modification | Average revenue precision |
| `COUNT(DISTINCT TID)` | Unique transaction counting |

---

## 🛠️ Tools

**Language:** SQL · **Database:** MySQL  
**Schema pattern:** Star Schema · **ETL pattern:** Staging → Fact → Aggregate → DW

---

## 🚀 How to Run

```bash
git clone https://github.com/AbyJoeJose/Data-Science-Portfolio.git
cd Sales-Data-Warehouse-SQL
```

```sql
-- In your MySQL client:

-- Step 1: Ensure source operational DB exists
USE josea_zagimore;

-- Step 2: Create staging and DW schemas
CREATE DATABASE josea_zagimore_staging;
CREATE DATABASE josea_zagimore_dw;

-- Step 3: Run the ETL script
SOURCE Sales_Data_Warehouse.sql;
```

---

## 📁 File Structure

```
Sales-Data-Warehouse-SQL/
├── Sales_Data_Warehouse.sql     # Complete ETL + schema script
└── README.md                    # This file
```

---

## 🔑 Key Takeaways

- **Intermediate fact table pattern** — using `IntermediateFact` as a staging buffer before resolving dimension surrogate keys is a production-grade ETL technique that keeps extraction logic separate from key lookup
- **Single Product_Dimension for both transaction types** — flagging rows with `ProductType = 'Sales'` or `'Rental'` avoids maintaining two separate dimension tables while still supporting type-specific filtering in queries
- **`LEFT(zip, 2)` for local detection** — an elegant SQL approach to determine local vs non-local customers without needing a geographic join
- **Incremental snapshot building** — creating the base snapshot then adding columns via `ALTER TABLE` and `UPDATE` from helper tables mirrors how production snapshots are refreshed in real ETL pipelines
- **`RevenueGenerated > 100` threshold** for high-value transactions — a simple but effective business rule embedded directly in the SQL
- **Composite primary keys** on aggregate and snapshot tables enforce data integrity without needing a single-column surrogate key

---

*Part of the [Data Science Portfolio](https://github.com/AbyJoeJose/Data-Science-Portfolio)*
