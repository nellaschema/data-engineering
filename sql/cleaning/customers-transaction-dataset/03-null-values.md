## transactions.csv


```sql
-- 3. Find null values 

SELECT *
FROM ftw.default.transactions
WHERE 
    customer_id IS NULL
    OR transaction_id IS NULL
    OR transaction_date IS NULL
    OR receipt_number IS NULL
    OR product_sku IS NULL
    OR product_brand IS NULL
    OR quantity IS NULL
    OR total_unit_price IS NULL
    OR retailer IS NULL
    OR branch IS NULL;
```
Result:

#### Records with NULL `product_brand`

| customer_id | transaction_id | receipt_date | transaction_date | receipt_number | product_sku | product_brand | quantity | total_unit_price | retailer | branch |
|---:|---:|---|---|---|---|---|---:|---:|---|---|
| 413 | 73 | 2024-07-06 15:28:28 | 2024-07-07T19:06:08.000+00:00 | 0000076477 | DP WHT VNGR 0.5GAL | NULL | 1 | 92 | Robinsons Supermarket | Robinsons Supermarket - Rockwell Drive |
| 527 | 95 | 2024-07-11 17:47:34 | 2024-07-13T05:43:51.000+00:00 | 136-009-00474132 | $UFC GFP/OL950ML | NULL | 1 | 122 | Puregold | PUREGOLD BAESA NOVALICHES QC |
| 527 | 95 | 2024-07-11 17:47:34 | 2024-07-13T05:43:51.000+00:00 | 136-009-00474132 | DP SOY SCE 200ML | NULL | 1 | 10.15 | Puregold | PUREGOLD BAESA NOVALICHES QC |`


```sql
-- Missing product_brand values: 3 records contained NULL product_brand values. The brand was inferred from the corresponding product_sku by comparing against other records with the same product/SKU naming convention. The missing values were standardized accordingly.

UPDATE ftw.default.transactions
SET product_brand =
    CASE 
        WHEN product_sku LIKE 'DP%' THEN 'Datu Puti'
        WHEN product_sku LIKE '%UFC%' THEN 'UFC'
    ELSE product_brand
        END
WHERE product_brand IS NULL;
```

Result:

| num_affected_rows |
|-------------------|
| 3                 |




## customers.csv


```sql
-- 3. Find null values
SELECT *
FROM ftw.default.customers
WHERE user_id IS NULL
   OR birthday IS NULL
   OR registered_date IS NULL;
```
Result:

`No rows returned`
