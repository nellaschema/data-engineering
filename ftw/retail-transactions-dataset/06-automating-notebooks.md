# Automating data cleaning up to the gold layer involves creating multiple tasks in `Jobs and Pipelines` uisng Notebooks.


## Create a Job

```text
-- Create one job and consolidate all notebooks.
ftw_project
├── raw
│   └── source_files (Volume)
├── bronze
│   ├── customers
│   ├── transactions
│   └── products
├── silver
│   ├── customers
│   ├── transactions
│   └── products
└── gold
    └── dashboard_data
```

`Run if dependencies`: all notebooks must be set to "All succeeded" because we want to make sure that all new data are ingested and cleaned properly before the next set of notebooks run.

# Automating a dashboard - this is not necessary but why not

```text
01. take the gold table
02. create a dashboard
03. update the Job created to include the automatic updating of the dashboard

# First run

After putting all four notebooks in the `retail_bronze_to_gold` Job, I tried running it and it showed the error:
```text
Task data_ingestion_customers failed with message: Workload failed, see run output for details. This caused all downstream tasks to get skipped.
```
I asked `Genie` what caused it and this was her response:
```text
Root Cause
The job failed with error [DELTA_FAILED_TO_MERGE_FIELDS] Failed to merge fields 'user_id' and 'user_id' in the data_ingestion_customers task.

This schema mismatch occurs because of inconsistent schema handling between Cell 2 and Cell 3 in the 01_retail_ingest_custom... notebook:

Cell 2 created the table ftw.02_bronze.retail_customers using read_files with inferSchema => true, which correctly typed the user_id column as int based on the CSV data.

Cell 3 uses COPY INTO without schema inference (inferSchema is not specified in FORMAT_OPTIONS). By default, COPY INTO reads all CSV columns as strings when schema inference is disabled.

When Delta Lake attempts to merge the string-typed user_id data from the CSV into the existing table's integer user_id column, it detects the type incompatibility and raises DELTA_FAILED_TO_MERGE_FIELDS.

I reached this conclusion by:

Retrieving the failed task run output showing the DELTA_FAILED_TO_MERGE_FIELDS error
Reading the notebook source to see the exact SQL in each cell
Observing that Cell 1 describes the CSV as having user_id: int, Cell 2 creates the table with inferSchema => true (preserving that int type), but Cell 3's COPY INTO lacks schema inference
```



### Tried running it again: 
Several failed runs were all about syntax. Genie helped point them out and I just corrected it.

**FIX**
Update Cell 3 of `01_retail_ingestion_customers` to include schema inference in the COPY INTO command:


```sql
-- NEW CSV file ingestion 
COPY INTO ftw.`02_bronze`.retail_customers
FROM '/Volumes/ftw/01_source_files/retail_customers'
FILEFORMAT = CSV
FORMAT_OPTIONS (
    'header' = 'true',
    'inferSchema' = 'true'
);
```

Update Cell 3 of `02_silver_to_gold` to include schema inference in the COPY INTO command:


```sql
-- NEW CSV file ingestion 
COPY INTO ftw.`02_bronze`.retail_transactions
FROM '/Volumes/ftw/01_source_files/retail_transactions'
FILEFORMAT = CSV
FORMAT_OPTIONS (
    'header' = 'true',
    'inferSchema' = 'true'
);
```

Update Cell 4 of `04_silver_to_gold`:


```sql
-- Row count check

SELECT COUNT(*) AS customers_gold_row_count,
       COUNT(DISTINCT user_id) AS distinct_customers
FROM ftw.`04_gold`.retail_customers_gold;
```

