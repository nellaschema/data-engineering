```sql
-- 7. Check the length of receipt numbers
SELECT
    LENGTH(receipt_number) AS receipt_number_length,
    COUNT(*) AS row_count
FROM ftw.default.transactions
WHERE receipt_number IS NOT NULL
GROUP BY LENGTH(receipt_number)
ORDER BY receipt_number_length;
```


Result:

| receipt_number_length | row_count |
| --------------------: | --------: |
|                     6 |        29 |
|                     7 |         6 |
|                     8 |        92 |
|                     9 |       388 |
|                    10 |     1,254 |
|                    11 |       232 |
|                    12 |        57 |
|                    13 |         4 |
|                    14 |       587 |
|                    15 |         4 |
|                    16 |       511 |
|                    18 |         1 |
|                    19 |         1 |
|                    36 |         1 |

```sql
-- Check if the long receipt numbers are associated with a certain branch
SELECT LENGTH(receipt_number) AS receipt_number_length, branch, retailer
FROM ftw.default.transactions
WHERE receipt_number IS NOT NULL
GROUP BY branch, receipt_number_length, retailer
ORDER BY branch
```

