### Move tables from one schema to another using the following code:

```sql
ALTER TABLE table_name
SET SCHEMA new_schema
```

#### Verify the database location:

```sql
SHOW TABLES IN new_schema
```

#### Verify the physical location:

```sql
DESCRIBE DETAIL new_schema.table_name
```
