# Employee Data Warehouse with SCD Type 2

An end-to-end Data Engineering project built on **Databricks** and **Delta Lake**, implementing **Slowly Changing Dimensions (SCD Type 2)** to track historical changes in employee data — department transfers, salary revisions, manager changes, and location changes — while preserving full audit history.

## Problem Statement

In real organizations, employee attributes change constantly (promotions, transfers, salary revisions). A naive pipeline that simply overwrites old data loses this history. This project solves that by implementing SCD Type 2, a standard data warehousing technique used to maintain a complete historical record of every change.

## Architecture

![Architecture Diagram](docs/architecture_diagram.png)

- **Bronze:** Raw employee and performance data generated using Python (Faker library), stored as Delta tables.
- **Silver:** Cleaned data — fixed inconsistent NULL representations, trimmed whitespace, validated records.
- **SCD Type 2 Table:** Tracks every version of an employee's record with `start_date`, `end_date`, and `is_current` flags.
- **Analytics:** SQL-based historical analysis using subqueries and window functions.

## Tech Stack

- **Databricks** (Community/Free Edition)
- **PySpark** — data generation and transformation
- **Delta Lake** — ACID-compliant storage, MERGE operations
- **SQL** — data cleaning, subqueries, window functions

## Key Features

1. **Data Quality Validation** — Correlated subqueries to detect duplicate and inconsistent employee records.
2. **SCD Type 2 Implementation** — Delta Lake `MERGE` statement with NULL-safe comparisons (`IS DISTINCT FROM`) to correctly detect changes even when fields contain NULL values.
3. **Historical Analytics:**
   - "As-of" queries — retrieve an employee's attributes as they were on any past date
   - Salary progression tracking using the `LAG()` window function
   - Department-wise current headcount analysis

## Sample Insight

When an employee is transferred or gets a raise, the old record is closed (`is_current = false`, `end_date` set) and a new record is created (`is_current = true`, `end_date = NULL`) — both versions remain queryable, enabling full historical reporting.

## Notable Technical Challenge

Initial MERGE logic used standard `!=` comparisons, which silently failed to detect changes in NULL fields (e.g., `manager_id` changing from `NULL` to a value), since SQL's three-valued logic evaluates `NULL != value` as `NULL`, not `TRUE`. Fixed by switching to `IS DISTINCT FROM`, which handles NULL comparisons correctly. This is documented as a lesson learned in the project notebook.

## Project Structure

```
Employee_SCD_Warehouse/
│
├── notebooks/
│   ├── 01_bronze_ingestion.py          Generates fake employee & performance data (Python + Faker)
│   └── 02_scd_pipeline_final.sql       Complete SQL pipeline: cleaning, SCD Type 2, analytics
│
├── docs/
│   └── architecture_diagram.png        Bronze -> Silver -> SCD -> Analytics flow (optional)
│
├── README.md                           Project overview, architecture, tech stack
└── requirements.txt                    faker (for local data generation)
```

## Setup

1. Import `01_bronze_ingestion.py` and `02_scd_pipeline_final.sql` into a Databricks workspace (or run locally with a PySpark environment).
2. Install dependencies: `pip install -r requirements.txt`
3. Run `01_bronze_ingestion.py` first to generate the Bronze layer tables.
4. Run `02_scd_pipeline_final.sql` to execute the full Silver -> SCD Type 2 -> Analytics pipeline.

## Author

**Yashashree Jena**
Siksha 'O' Anusandhan University
