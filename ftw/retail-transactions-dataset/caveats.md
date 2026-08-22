# 01. Incremental Ingestion and Upsert

## What We Currently Have

Our Bronze ingestion notebook already uses `COPY INTO`, so the Bronze part of the pipeline does not need to change.

```sql
-- NEW CSV file ingestion 
-- COPY INTO tracks processed files automatically to prevent duplicates
COPY INTO ftw.`02_bronze`.retail_customers
FROM '/Volumes/ftw/01_source_files/retail_customers'
FILEFORMAT = CSV
FORMAT_OPTIONS (
    'header' = 'true',
    'inferSchema' = 'true'
)
COPY_OPTIONS (
    'force' = 'false'  -- Only ingest files not previously processed
);``
```

This is appropriate for the **new-file ingestion** part of the pipeline.

The current flow is:

```text
Volume
   ↓
COPY INTO
   ↓
Bronze
```

`COPY INTO` tracks files that it has already ingested. Therefore, if another CSV is added to the `retail_customers` folder and the notebook is run again, the new file can be ingested without manually tracking which files were already processed.

## Important Distinction

`COPY INTO` handles **file-level ingestion**, but it does not perform an upsert based on `user_id`.

For example, suppose Bronze already contains:

```text
user_id | birthday   | registered_date
001     | 1990-01-01 | 2025-01-01
```

Then a new CSV arrives containing:

```text
user_id | birthday   | registered_date
001     | 1990-01-02 | 2025-01-01
002     | 1995-05-10 | 2026-08-20
```

The new records can be appended to Bronze:

```text
001 | 1990-01-01 | 2025-01-01
001 | 1990-01-02 | 2025-01-01
002 | 1995-05-10 | 2026-08-20
```

`COPY INTO` does not know that the second `001` is intended to replace the previous `001`.

## Pipeline We Are Trying to Understand

```text
NEW CSV
   ↓
Volume
   ↓
COPY INTO
   ↓
BRONZE
   ↓
Clean / prepare incoming records
   ↓
MERGE INTO
   ↓
SILVER
(one current row per user_id)
   ↓
GOLD
   ↓
Dashboard
```

## Questions for the Instructor

### 1. Bronze and COPY INTO

* Is using `COPY INTO` from the Volume the recommended approach for our Bronze ingestion?
* Does `COPY INTO` track processed files across scheduled job runs, or are there situations where we need to manage ingestion state ourselves?
* What happens if a file that was already ingested is modified and uploaded again?
* Should we keep the original CSV files in the Volume after they have been ingested?

### 2. Handling Updated Customer Records

If a new CSV contains an existing `user_id` with corrected information:

```text
Existing:
001 | 1990-01-01 | 2025-01-01

New:
001 | 1990-01-02 | 2025-01-01
```

* Should Bronze contain both versions?
* Should Bronze always be append-only?
* At what layer should the existing customer record be replaced or updated?
* Is `MERGE INTO` the appropriate approach for maintaining the current customer record in Silver?

### 3. Identifying the Latest Record

If the same `user_id` appears multiple times in an incoming file:

```text
001 | 1990-01-01 | ...
001 | 1990-01-02 | ...
```

* How should we determine which record is the latest?
* Should we add an ingestion timestamp when loading the data?
* Is the file's modified/created timestamp sufficient, or should we create our own ingestion timestamp?
* What is the recommended way to deduplicate the incoming records before a `MERGE`?

### 4. Silver Layer

* Should our Silver customer table maintain exactly one current record per `user_id`?
* Should the Silver table be updated using `MERGE INTO` rather than `CREATE OR REPLACE TABLE`?
* Should the cleaning transformations happen before the `MERGE`, or should we merge first and clean afterward?
* If a customer record is corrected in a new CSV, should the old value simply be overwritten, or should we maintain historical versions?

### 5. Gold Layer

* If Silver already maintains the current customer records, is it acceptable to rebuild the Gold customer table from Silver?
* At what point would we need incremental processing in Gold instead of simply rebuilding it?
* For the joined customer-transactions Gold table, should we rebuild it whenever Silver changes, or is there a preferred incremental approach?

### 6. Overall Pipeline Design

For our project, would this be a reasonable architecture?

```text
Volume
  ↓
COPY INTO
  ↓
Bronze
  ↓
Clean + validate incoming data
  ↓
MERGE
  ↓
Silver
  ↓
Gold
  ↓
Dashboard
```

More specifically, we want to understand the distinction between:

```text
File-level incremental ingestion
        vs.
Record-level upsert
```

Our current understanding is that `COPY INTO` solves the first problem, while something such as `MERGE INTO` may be needed for the second. We would like to confirm whether this is the appropriate design for this project.
