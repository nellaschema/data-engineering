
### 01. Update Cell 3 of `01_retail_ingestion_customers` to include schema inference in the COPY INTO command:


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

### 02. Update Cell 3 of `02_silver_to_gold` to include schema inference in the COPY INTO command:


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

### 02. Update Cell 4 of `04_silver_to_gold`:


```sql
-- Row count check

SELECT COUNT(*) AS customers_gold_row_count,
       COUNT(DISTINCT user_id) AS distinct_customers
FROM ftw.`04_gold`.retail_customers_gold;
```

