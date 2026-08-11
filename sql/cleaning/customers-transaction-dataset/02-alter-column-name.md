## transactions.csv

### First column name was written as `# customer_id`

```sql
-- 1. Correct column name:
ALTER TABLE ftw.default.transactions
RENAME COLUMN `# customer_id` TO customer_id;
```
Result:

`OK`
