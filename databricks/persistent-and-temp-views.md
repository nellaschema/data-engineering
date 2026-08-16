### Sometimes we might want to create a view from a table for easy referencing. 

```sql
CREATE VIEW view_name AS
SELECT relevant_column1, relevant_column2
FROM table_name
```

#### Use `CREATE TEMP VIEW` instead if you're only gonna use the view in that current session. `CREATE VIEW` makes a persistent view, which means it won't be automatically dropped between sessions until you drop it using the command `DROP VIEW`.

#### To check the current active views, use 
```sql 
SHOW VIEWS IN schema_name
```
