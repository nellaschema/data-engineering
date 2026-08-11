## transactions.csv


```sql
-- 4. Find product_sku with weird characters/prefixes.
SELECT
    product_sku,
    COUNT(*) AS row_count
FROM ftw.default.transactions
WHERE product_sku IS NOT NULL
  AND product_sku RLIKE '^[^A-Za-z0-9]'
        -- looks for SKUs that start with something other than a letter or number.
GROUP BY product_sku
ORDER BY row_count DESC;
```
Result:

#### Records with starting with characters which are not letters or number

| product_sku                 | row_count |
| --------------------------- | --------: |
| `$UFC GFP/OL950ML`          |        80 |
| `<< UFC T/A B/KET32`        |        62 |
| `$UFC GFP/OL1LSUP`          |        58 |
| `$UFC GF CNLA OIL1`         |        29 |
| `$UFC T/A B/KTCHP1K`        |        17 |
| `<< DP W/VIN 385ML`         |        15 |
| `*MDF_UFC CREAM OF MUSHROO` |        11 |
| `*MDF_UFC CHICKEN AND CORN` |        10 |
| `$UFC GF CAOLA 2L`          |        10 |
| `*MDF_UFC COCONUT CREAM400` |         9 |
| `$UFC B/KTCHP H&S32`        |         8 |
| `<< DP SOYSCE 385ML`        |         7 |
| `$UFC GFP/OL 2L`            |         7 |
| `*MDF_UFC BKTCHP REG 1KG/1` |         6 |
| `$UFC GF SOA OIL 1`         |         6 |
| `<<DP SOYSCE 385ML`         |         6 |
| `*MDF_UFC SCE HOT & SPICY`  |         6 |
| `<<DP W/VIN 385ML`          |         5 |
| `*MDF UFC CORN WHL KERNEL`  |         5 |
| `*MDF_UFC BKTCHP POUCH 320` |         5 |
| `$<<UFCGFOIL485ML`          |         4 |
| `*MDF_UFC BKTCHP SVERSPACK` |         4 |
| `<<PAPA B/KTCHP320G`        |         4 |
| `*MDF_UFC SCE SWEET CHILI`  |         3 |
| `$ << UFCGFOIL485ML`        |         2 |
| `$UFC 2N1 SS 1K`            |         2 |
| `$UFC GFCOR OIL1L`          |         2 |
| `*MDF_UFC BKTCHP REG 530G/` |         2 |
| `<<SS CANE VIN350ML`        |         2 |
| `*MDF_UFC RICEMX FUNCHW ME` |         1 |
| `<<DP Oyster 170g`          |         1 |
| `*MDF_UFC BKTCHP RICH BT 3` |         1 |
| `*MDF_UFC SCE SWEETCHILI 3` |         1 |
| `$UFC GFP/OL930ML`          |         1 |
| `*MDF_UFC BKTCHP REG 320GR` |         1 |
|                            |           |


```sql
-- Perform comparison first  
SELECT 
    product_sku AS old_sku,
    CASE
        WHEN product_sku RLIKE '^\\$ << ' 
            THEN REGEXP_REPLACE(product_sku, '^\\$ << ', '')
        WHEN product_sku RLIKE '^\\$<<' 
            THEN REGEXP_REPLACE(product_sku, '^\\$<<', '')
        WHEN product_sku RLIKE '^\\$' 
            THEN REGEXP_REPLACE(product_sku, '^\\$', '')
        WHEN product_sku RLIKE '^<< ' 
            THEN REGEXP_REPLACE(product_sku, '^<< ', '')
        WHEN product_sku RLIKE '^<<' 
            THEN REGEXP_REPLACE(product_sku, '^<<', '')
        WHEN product_sku RLIKE '^\\*MDF_' 
            THEN REGEXP_REPLACE(product_sku, '^\\*MDF_', '')
        WHEN product_sku RLIKE '^\\*MDF ' 
            THEN REGEXP_REPLACE(product_sku, '^\\*MDF', '')
        ELSE product_sku
    END AS new_sku
FROM ftw.default.transactions
WHERE product_sku IS NOT NULL
  AND product_sku RLIKE '^[^A-Za-z0-9]';
    -- looks for SKUs that start with something other than a letter or number.
```

Result:

| old_sku                         | new_sku                    |
| ------------------------------- | -------------------------- |
| `$UFC GFP/OL950ML`              | `UFC GFP/OL950ML`          |
| `<< UFC T/A B/KET32`            | `UFC T/A B/KET32`          |
| `$UFC GFP/OL1LSUP`              | `UFC GFP/OL1LSUP`          |
| `$UFC GF CNLA OIL1`             | `UFC GF CNLA OIL1`         |
| `$UFC T/A B/KTCHP1K`            | `UFC T/A B/KTCHP1K`        |
| `<< DP W/VIN 385ML`             | `DP W/VIN 385ML`           |
| `*MDF_UFC CREAM OF MUSHROO`     | `UFC CREAM OF MUSHROO`     |
| `*MDF_UFC CHICKEN AND CORN`     | `UFC CHICKEN AND CORN`     |
| `$UFC GF CAOLA 2L`              | `UFC GF CAOLA 2L`          |
| `*MDF_UFC COCONUT CREAM400`     | `UFC COCONUT CREAM400`     |
| `$UFC B/KTCHP H&S32`            | `UFC B/KTCHP H&S32`        |
| `<< DP SOYSCE 385ML`            | `DP SOYSCE 385ML`          |
| `$UFC GFP/OL 2L`                | `UFC GFP/OL 2L`            |
| `*MDF_UFC BKTCHP REG 1KG/1`     | `UFC BKTCHP REG 1KG/1`     |
| `$UFC GF SOA OIL 1`             | `UFC GF SOA OIL 1`         |
| `<<DP SOYSCE 385ML`             | `DP SOYSCE 385ML`          |
| `*MDF_UFC SCE HOT & SPICY`      | `UFC SCE HOT & SPICY`      |
| `<<DP W/VIN 385ML`              | `DP W/VIN 385ML`           |
| `*MDF UFC CORN WHL KERNEL`      | `UFC CORN WHL KERNEL`      |
| `*MDF_UFC BKTCHP POUCH 320`     | `UFC BKTCHP POUCH 320`     |
| `$<<UFCGFOIL485ML`              | `UFCGFOIL485ML`            |
| `*MDF_UFC BKTCHP SVERSPACK`     | `UFC BKTCHP SVERSPACK`     |
| `<<PAPA B/KTCHP320G`            | `PAPA B/KTCHP320G`         |
| `*MDF_UFC SCE SWEET CHILI`      | `UFC SCE SWEET CHILI`      |
| `$ << UFCGFOIL485ML`            | `UFCGFOIL485ML`            |
| `$UFC 2N1 SS 1K`                | `UFC 2N1 SS 1K`            |
| `$UFC GFCOR OIL1L`              | `UFC GFCOR OIL1L`          |
| `*MDF_UFC BKTCHP REG 530G/`     | `UFC BKTCHP REG 530G/`     |
| `<<SS CANE VIN350ML`            | `SS CANE VIN350ML`         |
| `*MDF_UFC RICEMX FUNCHW ME`     | `UFC RICEMX FUNCHW ME`     |
| `<<DP Oyster 170g`              | `DP Oyster 170g`           |
| `*MDF_UFC BKTCHP RICH BT 3`     | `UFC BKTCHP RICH BT 3`     |
| `*MDF_UFC SCE SWEETCHILI 3`     | `UFC SCE SWEETCHILI 3`     |
| `$UFC GFP/OL930ML`              | `UFC GFP/OL930ML`          |
| `*MDF_UFC BKTCHP REG 320GR`     | `UFC BKTCHP REG 320GR`     |
| `*MDF_UFC SPCS CHILI PWDR`      | `UFC SPCS CHILI PWDR`      |
| `*MDF_UFC SPSAU SCE1KG+PST`     | `UFC SPSAU SCE1KG+PST`     |
| `*MDF_UFC SPSAU SWTFILSTYL 1kg` | `UFC SPSAU SWTFILSTYL 1kg` |
| `$UFC GFPOI3.785L`              | `UFC GFPOI3.785L`          |
| `<< PAPA B/KTCHP320G`           | `PAPA B/KTCHP320G`         |




```sql
-- Update the product SKUs 
UPDATE ftw.default.transactions 
SET product_sku =
    CASE
        WHEN product_sku RLIKE '^\\$ << ' 
            THEN REGEXP_REPLACE(product_sku, '^\\$ << ', '')
        WHEN product_sku RLIKE '^\\$<<' 
            THEN REGEXP_REPLACE(product_sku, '^\\$<<', '')
        WHEN product_sku RLIKE '^\\$' 
            THEN REGEXP_REPLACE(product_sku, '^\\$', '')
        WHEN product_sku RLIKE '^<< ' 
            THEN REGEXP_REPLACE(product_sku, '^<< ', '')
        WHEN product_sku RLIKE '^<<' 
            THEN REGEXP_REPLACE(product_sku, '^<<', '')
        WHEN product_sku RLIKE '^\\*MDF_' 
            THEN REGEXP_REPLACE(product_sku, '^\\*MDF_', '')
        WHEN product_sku RLIKE '^\\*MDF ' 
            THEN REGEXP_REPLACE(product_sku, '^\\*MDF', '')
        ELSE product_sku
    END
WHERE product_sku IS NOT NULL
  AND product_sku RLIKE '^[^A-Za-z0-9]';
```

Result:
|num_affected_rows|
|-----------------|
|399|
