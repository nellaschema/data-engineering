# Becoming a Data Engineer

A learning project documenting my transition into data engineering through hands-on projects, mainly using Databricks, SQL, Python, and GitHub.

I'm building this repository as I learn, so it includes notes, experiments, errors, fixes, and decisions along the way.

---

## Current Project

### Retail Data Engineering Pipeline

The current project is a retail data pipeline using two source datasets:

- `retail_customers.csv`
- `retail_transactions.csv`

The goal is to take raw CSV files, ingest them into Databricks, clean and validate the data, transform the datasets, and prepare them for reporting/dashboard use.

### Pipeline

```text
CSV Files
    ↓
Databricks Volume
    ↓
Bronze
    ↓
Silver
    ↓
Gold
    ↓
Dashboard
```

---

## Architecture

```text
ftw
│
├── 01_source_files
│   └── retail_customers/
│       └── *.csv
│   └── retail_transactions/
│       └── *.csv
│
├── 02_bronze
│   ├── retail_customers
│   └── retail_transactions
│
├── 03_silver
│   ├── retail_customers_cleaned
│   └── retail_transactions_cleaned
│
└── 04_gold
    ├── retail_customers_gold
    ├── retail_transactions_gold
    └── customer_transactions_joined
```

### Layer responsibilities

| Layer | Purpose |
|---|---|
| Source / Volume | Stores the raw CSV files |
| Bronze | Stores ingested source data with minimal transformation |
| Silver | Cleans, validates, and standardizes the data |
| Gold | Prepares data for analysis, reporting, and dashboards |

---

## Data Ingestion

The source CSVs are stored in a Databricks Volume.

For incremental file ingestion, I'm using `COPY INTO`.

Example:

```sql
COPY INTO ftw.`02_bronze`.retail_customers
FROM '/Volumes/ftw/01_source_files/retail_customers'
FILEFORMAT = CSV
FORMAT_OPTIONS (
    'header' = 'true'
);
```

The idea is:

```text
New CSV
   ↓
Volume
   ↓
COPY INTO
   ↓
Bronze
```

`COPY INTO` tracks files that have already been ingested, so the ingestion notebook can be run again when new CSV files are added.

One thing I'm still learning is the distinction between:

```text
File-level incremental ingestion
        vs.
Record-level upsert
```

For example, if a new CSV contains an existing `user_id`, `COPY INTO` does not automatically know that the new record should replace the previous record.

Questions around incremental ingestion and `MERGE INTO` are documented in the project notes.

---

# Silver Layer

The Silver layer contains cleaned and validated versions of the Bronze datasets.

## Retail Transactions

Main cleaning steps include:

1. Standardizing inconsistent date formats
2. Checking receipt/transaction date ordering
3. Cleaning malformed SKUs
4. Standardizing branch names
5. Removing invalid zero/negative transaction values
6. Handling missing product brands
7. Investigating duplicate transaction IDs
8. Distinguishing legitimate receipt line items from true duplicates
9. Calculating unit price
10. Calculating median and standard deviation of SKU prices
11. Flagging suspicious price outliers
12. Validating the final cleaned dataset

The cleaning process uses temporary views:

```text
Bronze
  ↓
step1
  ↓
step2
  ↓
step3
  ↓
...
  ↓
Final Silver Table
```

This allows each transformation to be inspected before moving to the next step.

## Retail Customers

Main cleaning/validation steps include:

1. Checking missing values
2. Checking duplicate `user_id`
3. Checking fully duplicated records
4. Standardizing date types
5. Calculating age at registration
6. Flagging implausible ages
7. Checking whether registration occurred before birth
8. Validating the final Silver table

---

# Gold Layer

The Gold layer prepares the cleaned data for reporting and dashboard use.

Three tables are currently planned:

```text
04_gold
├── retail_customers_gold
├── retail_transactions_gold
└── customer_transactions_joined
```

The customer and transaction Gold tables preserve their natural grain.

```text
retail_customers_gold
→ one row per customer

retail_transactions_gold
→ one row per transaction line item

customer_transactions_joined
→ one row per transaction line item with customer information
```

The joined table is intended for BI/dashboard use where customer and transaction attributes are needed together.

---

# Project Notebooks

Current notebook structure:

```text
notebooks/
│
├── 01_retail_ingestion_customers
├── 02_retail_ingestion_transactions
├── 03_retail_bronze_to_silver
└── 04_retail_silver_to_gold
```

The exact notebook structure may change as the pipeline becomes more automated.

---

# Data Quality Approach

I'm trying to distinguish between data that should be:

```text
CORRECTED
    ↓
FLAGGED
    ↓
REMOVED
```

Not every unusual value should automatically be deleted.

For example:

- A malformed SKU can be corrected.
- An unusual price can be flagged for investigation.
- A zero-quantity transaction may be removed if it represents an invalid transaction.
- Multiple rows sharing a receipt number are not necessarily duplicates because one receipt can contain multiple products.

The goal is to make the cleaning decisions explicit rather than silently modifying the data.

---

# Validation

Each layer is validated before moving to the next layer.

Examples:

```text
Raw row count
        ↓
Clean row count
        ↓
Missing values
        ↓
Invalid values
        ↓
Duplicates
        ↓
Business-rule checks
```

For the Gold layer, the join is also validated to make sure transaction rows are not unintentionally dropped or duplicated.

---

# Automation

The eventual goal is to make the pipeline repeatable:

```text
New CSV
   ↓
Volume
   ↓
Scheduled ingestion
   ↓
Bronze
   ↓
Cleaning
   ↓
Silver
   ↓
Transformation / Join
   ↓
Gold
   ↓
Dashboard
```

I'm currently learning how to handle incremental data, updated records, and upserts properly.

---

# What I'm Learning

This project is helping me practice:

- SQL
- Databricks
- Unity Catalog
- Volumes
- Delta tables
- `COPY INTO`
- Temporary views
- CTEs
- Window functions
- `MERGE INTO`
- Data cleaning
- Data validation
- Data quality checks
- Medallion architecture
- ETL/ELT pipelines
- Job scheduling
- Git/GitHub
- Dashboard preparation

---


# Tools

Current tools:

```text
Databricks
SQL
Python
GitHub
Google Sheets
```


---

# Progress

This repository is a work in progress.

I'm documenting the process as I learn rather than only documenting the final solution. That means some notebooks and notes may contain failed approaches, debugging, and questions that were later resolved.

The point of the repository is to track the progression from:

```text
Learning individual SQL commands
            ↓
Understanding data cleaning
            ↓
Building transformations
            ↓
Understanding Bronze / Silver / Gold
            ↓
Building repeatable pipelines
            ↓
Learning automation and orchestration
```
