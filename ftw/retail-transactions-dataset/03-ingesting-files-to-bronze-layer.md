The goal is to build an automated process where we can add new CSV files to a Databricks Volume and have the data automatically loaded into the appropriate Bronze table.

Our overall flow will be:

```text
CSV files
   ↓
Databricks Volume
   ↓
Bronze
   ↓
Silver
   ↓
Gold
```

We want to keep the raw files separate from the processed tables so that we can trace where the data came from and easily process updated or new files later.

## File Structure

We will organize the source files in the Volume by dataset:

```text
ftw
└── 01_source_files
    ├── retail_customers/
    │   └── customers.csv
    │
    └── retail_transactions/
        └── transactions.csv
```

The folder tells us which dataset the files belong to. This means the filename does not have to follow one exact naming convention.

For example, any new customer files should go into:

```text
/Volumes/ftw/01_source_files/retail_customers/
```

## Initial Bronze Load

If the Bronze table does not exist yet, We can create it and load the initial data using `read_files()`:

```sql
CREATE TABLE IF NOT EXISTS ftw.`02_bronze`.retail_customers
AS
SELECT *
FROM read_files(
  '/Volumes/ftw/01_source_files/retail_customers',
  format => 'csv',
  header => true,
  inferSchema => true
);
```

This creates the table and loads the data at the same time.

```text
CSV in Volume
      ↓
read_files()
      ↓
Bronze Table
```

## Loading New Files

Once the Bronze table already exists, We can use `COPY INTO` to load additional files:

```sql
COPY INTO ftw.`02_bronze`.retail_customers
FROM '/Volumes/ftw/01_source_files/retail_customers'
FILEFORMAT = CSV
FORMAT_OPTIONS (
  'header' = 'true'
);
```

So the distinction is:

```text
CREATE TABLE AS SELECT + read_files()
→ create the Bronze table and perform the initial load

COPY INTO
→ load additional files into an existing Bronze table
```

For example, if later we add:

```text
retail_customers/
├── customers_january.csv
├── customers_february.csv
└── customers_march.csv
```

We can run the same `COPY INTO` statement again to ingest the new file.

## Automation

Once the manual process works, We can put the recurring ingestion code into a Databricks Job.

The eventual flow will be:

```text
New CSV added to Volume
          ↓
      Job trigger
          ↓
   Ingestion notebook
          ↓
      COPY INTO
          ↓
     Bronze table
          ↓
   Bronze → Silver
          ↓
   Silver → Gold
```

Takeaway: The **notebook contains the instructions**, while the **Job controls when those instructions run**.

## Notebook Structure

For each dataset, I plan to keep the ingestion separate:

```text
01_ingest_customers
02_ingest_transactions
03_bronze_to_silver
04_silver_to_gold
```
This keeps the datasets separate in bronze layer. We can then join the customer and transaction tables during the Silver transformation rather than combining the raw datasets immediately.

