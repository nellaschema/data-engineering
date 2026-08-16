### In Databricks, an `unmanaged` table means the data is stored somewhere else. We can do this by:

```sql
CREATE TABLE unmanaged_table
USING DELTA
LOCATION 's3://test//unmanaged_table_data
```

#### Removing this table in Databricks using ```DROP TABLE``` will not remove the original data from its source.
