## transactions.csv


```sql
-- 5. Check data for mixed date formats.
SELECT DISTINCT receipt_date
FROM ftw.default.transactions
LIMIT 100;
```
Result:

#### Some records with mixed date formats such as:
 - `YYYY-MM-DD HH:MM:SS`
 - `MM/DD/YYYY HH:MM`

| receipt_date          |
| --------------------- |
| `2024-07-29 15:44:00` |
| `2024-07-27 14:11:31` |
| `2024-08-03 10:22:00` |
| `2024-08-02 19:10:00` |
| `2024-08-03 11:23:33` |
| `2024-08-18 10:57:00` |
| `2024-08-26 18:23:57` |
| `10/06/2024 19:25`    |
| `10/09/2024 11:07`    |
| `10/16/2024 9:00`     |
| `10/26/2024 15:46`    |
| `10/31/2024 11:50`    |
| `10/20/2024 13:42`    |
| `09/25/2024 16:49`    |
| `11/14/2024 10:02`    |
| `11/15/2024 11:13`    |
| `11/23/2024 16:05`    |
| `11/26/2024 11:18`    |
| `11/26/2024 16:46`    |
| `12/20/2024 17:38`    |



```sql
-- 5. Check if date formats can be corrected
SELECT receipt_date,
    CASE
        WHEN receipt_date RLIKE '^[0-9]{4}-[0-9]{2}-[0-9]{2}' 
                                --regex used to recognize strings that start with a date in YYYY-MM-DD format.
            THEN try_to_timestamp(receipt_date, 'yyyy-MM-dd H:mm:ss')
        WHEN receipt_date RLIKE '^[0-9]{2}/[0-9]{2}/[0-9]{4}'
                                  --regex used to recognize strings with slash format MM/DD/YYYY.
            THEN try_to_timestamp(receipt_date, 'MM/dd/yyyy H:mm')
                                    -- to_timestamp means we tell databricks how to interpret it
    END AS receipt_date_clean
FROM ftw.default.transactions;
```

Result:

#### Successful conversion to ISO style:

| receipt_date          | receipt_date_clean              |
| --------------------- | ------------------------------- |
| `2024-06-03 18:27:00` | `2024-06-03T18:27:00.000+00:00` |
| `2024-06-03 18:27:00` | `2024-06-03T18:27:00.000+00:00` |
| `2024-06-26 18:09:48` | `2024-06-26T18:09:48.000+00:00` |
| `2024-06-30 17:19:32` | `2024-06-30T17:19:32.000+00:00` |
| `2024-06-30 17:19:32` | `2024-06-30T17:19:32.000+00:00` |
| `2024-07-06 15:28:28` | `2024-07-06T15:28:28.000+00:00` |
| `2024-07-06 15:28:28` | `2024-07-06T15:28:28.000+00:00` |


-- seems like slash formats were not recognized. see next query



```sql
-- 5. converting slash formats
SELECT
    receipt_date,
    try_to_timestamp(receipt_date, 'MM/dd/yyyy H:mm') AS receipt_date_clean
FROM ftw.default.transactions
WHERE receipt_date RLIKE '^[0-9]{2}/[0-9]{2}/[0-9]{4}'
LIMIT 20;
```


Result:

#### Successful conversion to ISO style:

| receipt_date       | receipt_date_clean              |
| ------------------ | ------------------------------- |
| `07/10/2024 13:20` | `2024-07-10T13:20:00.000+00:00` |
| `07/10/2024 13:20` | `2024-07-10T13:20:00.000+00:00` |
| `10/09/2024 19:20` | `2024-10-09T19:20:00.000+00:00` |
| `10/01/2024 8:37`  | `2024-10-01T08:37:00.000+00:00` |
| `10/01/2024 8:37`  | `2024-10-01T08:37:00.000+00:00` |
| `09/26/2024 8:34`  | `2024-09-26T08:34:00.000+00:00` |
| `09/10/2024 15:49` | `2024-09-10T15:49:00.000+00:00` |
| `10/10/2024 9:48`  | `2024-10-10T09:48:00.000+00:00` |
| `10/10/2024 9:48`  | `2024-10-10T09:48:00.000+00:00` |
| `10/03/2024 14:21` | `2024-10-03T14:21:00.000+00:00` |
| `09/28/2024 13:08` | `2024-09-28T13:08:00.000+00:00` |
| `10/10/2024 16:26` | `2024-10-10T16:26:00.000+00:00` |
| `10/10/2024 18:04` | `2024-10-10T18:04:00.000+00:00` |
| `10/10/2024 18:04` | `2024-10-10T18:04:00.000+00:00` |
| `10/11/2024 8:13`  | `2024-10-11T08:13:00.000+00:00` |
| `10/09/2024 15:43` | `2024-10-09T15:43:00.000+00:00` |




```sql
-- 5. Create a receipt_date_clean column
ALTER TABLE ftw.default.transactions
ADD COLUMNS (receipt_date_clean TIMESTAMP);
```


Result:

#### OK:

```sql
-- Update the values

UPDATE ftw.default.transactions
SET receipt_date_clean =
    CASE
        WHEN receipt_date RLIKE '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
            THEN try_to_timestamp(receipt_date, 'yyyy-MM-dd H:mm:ss')

        WHEN receipt_date RLIKE '^[0-9]{2}/[0-9]{2}/[0-9]{4}'
            THEN try_to_timestamp(receipt_date, 'MM/dd/yyyy H:mm')

        ELSE NULL
    END
WHERE receipt_date IS NOT NULL;
```


Result:

|num_affected_rows|
|-----------------|
|3167|



