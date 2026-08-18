### We have two source CSVs in the `source_files` Volume:

* `customers.csv`
* `transactions.csv`

Even though these datasets will eventually be joined, we should ingest them separately into the Bronze layer.

```text
customers.csv
    ↓
01_ingest_customers
    ↓
bronze.customers

transactions.csv
    ↓
02_ingest_transactions
    ↓
bronze.transactions
```

After both datasets are in Bronze, we can join them during the Silver transformation:

```text
bronze.customers
        +
bronze.transactions
        ↓
       JOIN
        ↓
silver.retail_sales_transactions
```

Proposed notebook structure:

```text
01_ingest_customers
02_ingest_transactions
03_bronze_to_silver
04_silver_to_gold
```

The Job can orchestrate the notebooks. The two ingestion tasks can run independently, while `03_bronze_to_silver` should run only after both Bronze tables are ready.

Reason for keeping them separate: Bronze should preserve the source datasets independently, while joining and business transformations belong in the Silver layer.
