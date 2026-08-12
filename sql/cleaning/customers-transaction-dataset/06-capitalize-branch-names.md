```sql
-- 6. Set all as upper
UPDATE ftw.default.transactions
SET branch = UPPER(branch)
WHERE branch IS NOT NULL;
```


Result:

|num_affected_rows|
|-----------------|
|3167|
