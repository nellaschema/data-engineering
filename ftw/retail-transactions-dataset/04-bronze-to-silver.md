# Retail Data Cleaning

## Goal

Clean the `retail_transactions` and `retail_customers` datasets from the Bronze layer and create validated Silver tables.

The cleaning process uses temporary views for each step so we can inspect and validate the data before creating the final Silver tables.

```text
Bronze
├── retail_transactions
└── retail_customers
        ↓
retail_bronze_to_silver
        ↓
Silver
├── retail_transactions_clean
└── retail_customers_cleaned
```

---

# 1. Retail Transactions

Source:

```text
ftw.`02_bronze`.retail_transactions
```

Target:

```text
ftw.`03_silver`.retail_transactions_clean
```

## Step 0: Check Raw Data

```sql
-- Quick look at the raw row count before any cleaning
SELECT COUNT(*)
FROM ftw.`02_bronze`.retail_transactions
```

Before cleaning, inspect the raw date values and identify formats that may not parse correctly.

## Step 1: Fix Receipt Dates

Some `receipt_date` values use different formats. `COALESCE()` tries the expected formats until one successfully parses.

```sql
-- STEP 1: Fix inconsistent receipt_date formats
-- Some receipt_date values are stored as 'M/D/YY H:MM' instead of the standard 'yyyy-MM-dd HH:mm:ss' used elsewhere, so they fail to parse.
-- COALESCE = try the standard format first, then falls back to the short format, so both patterns end up as a proper timestamp.
CREATE OR REPLACE TEMP VIEW step1_fixed_dates AS
SELECT
  *,
  COALESCE(
    TRY_TO_TIMESTAMP(receipt_date, 'yyyy-MM-dd HH:mm:ss'),
    TRY_TO_TIMESTAMP(receipt_date, 'M/d/yy H:mm')
  ) AS receipt_date_clean,
  TRY_TO_TIMESTAMP(transaction_date, 'yyyy-MM-dd HH:mm:ss') AS transaction_date_clean
FROM ftw.`02_bronze`.retail_transactions;
```

Validate:

```sql
-- Check: How many receipt_date values still fail to parse after the fix?
SELECT COUNT(*) AS still_unparsed_receipt_dates
FROM step1_fixed_dates
WHERE receipt_date_clean IS NULL;
-- Result: 137 null values ---- WHAT TO DO? idk yet
```

## Step 2: Check Receipt and Transaction Date Order

A receipt date occurring after the transaction date is logically questionable, so we must flag it for review.

```sql
-- STEP 2: Flag / drop rows where receipt_date is after transaction_date
-- A receipt logged before the transaction it belongs to is logically impossible, so we flag these for review rather than silently dropping them (swap the WHERE clause below to actually exclude them).
CREATE OR REPLACE TEMP VIEW step2_flagged_date_order AS
SELECT *,
  CASE
    WHEN receipt_date_clean > transaction_date_clean THEN TRUE
    ELSE FALSE
  END AS is_receipt_after_transaction
FROM step1_fixed_dates;
```

Check:

```sql
SELECT COUNT(*) AS impossible_date_order_rows
FROM step2_flagged_date_order
WHERE is_receipt_after_transaction = 'true';
-- Result: 65 rows
```

Inspect the affected rows:

```sql
SELECT
    transaction_id,
    receipt_number,
    receipt_date_clean,
    transaction_date_clean,
    unix_timestamp(receipt_date_clean)
      - unix_timestamp(transaction_date_clean) AS gap_seconds
FROM step2_flagged_date_order
WHERE is_receipt_after_transaction = 'true'
ORDER BY gap_seconds DESC;
```

These rows should be investigated before deciding whether to correct, flag, or remove them.

## Step 3: Clean Product SKU

Remove malformed prefixes/characters

```sql
-- STEP 3: Clean malformed product_sku values
-- Strip stray leading '$' and '×' (multiplication sign) characters that got mixed into the SKU text, and trim whitespace.
CREATE OR REPLACE TEMP VIEW step3_clean_sku AS
SELECT
    *,
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
    END AS product_sku_clean
FROM step2_flagged_date_order;
```

Validate:

```sql
-- Confirm no more malformed SKU prefixes remain in the cleaned SKU column
SELECT COUNT(*) AS remaining_malformed_skus
FROM step3_clean_sku
WHERE product_sku_clean RLIKE '^\\$|^<<|^\\*MDF';
-- Result: 0
```

## Step 4: Standardize Branch Names

Standardize capitalization, whitespace, and known spelling errors.

```sql
-- STEP 4: Standardize branch naming
-- Normalize casing (Title Case) and collapse extra whitespace so the same physical branch isn't split across multiple spellings/casings.
CREATE OR REPLACE TEMP VIEW step4_clean_branch AS
SELECT
  *,
  INITCAP(TRIM(REGEXP_REPLACE(branch, '\\s+', ' '))) AS branch_clean
FROM step3_clean_sku;
```

Validate:

```sql
-- Check how many distinct branch names remain after standardization (compare against the original 588 to measure the reduction)
SELECT COUNT(DISTINCT branch_clean) AS distinct_branches_after_cleanup
FROM step4_clean_branch;
-- branch names are consistently formatted if the number of rows didnt change
```

## Step 5: Remove Invalid Zero Values

```sql
-- STEP 5: Remove rows with zero quantity or zero total_unit_price. These represent invalid, non-purchase transactions.
CREATE OR REPLACE TEMP VIEW step5_remove_zero_values AS
SELECT *
FROM step4_clean_branch
WHERE quantity > 0
  AND total_unit_price > 0;
```

Validate:

```sql
-- Confirm the zero-value rows are gone
SELECT COUNT(*) AS remaining_zero_value_rows
FROM step5_remove_zero_values
WHERE quantity = 0 OR total_unit_price = 0;
-- Result: 0
```

## Step 6: Handle Missing Product Brand

Missing brands should be resolved using a reliable SKU-to-brand mapping where possible. If no reliable mapping exists, use `UNKNOWN`.

```sql
-- STEP 6: Handle missing product_brand
-- Infer missing brands from the cleaned product_sku where the SKU consistently maps to a single known brand. Use 'UNKNOWN' when no reliable SKU-based mapping exists.
CREATE OR REPLACE TEMP VIEW sku_brand_mapping AS
SELECT
    product_sku_clean,
    MAX(product_brand) AS product_brand
FROM step5_remove_zero_values
WHERE product_brand IS NOT NULL
GROUP BY product_sku_clean;
```

```sql
-- STEP 6: Handle missing product_brand
-- Infer missing brands from the cleaned product_sku where the SKU consistently maps to a single known brand. Use 'UNKNOWN' when no reliable SKU-based mapping exists.
CREATE OR REPLACE TEMP VIEW step6_fill_brand AS
SELECT
    s.* EXCEPT (product_brand),
    COALESCE(s.product_brand, m.product_brand, 'UNKNOWN') AS product_brand_clean
FROM step5_remove_zero_values s
LEFT JOIN sku_brand_mapping m
    ON s.product_sku_clean = m.product_sku_clean;
```

## Step 7: Check Transaction ID Duplicates

Repeated `transaction_id` values do not automatically mean duplicate records. A transaction can contain multiple product lines.

First inspect them:

```sql
-- STEP 7: Flag duplicate receipt_number for review
-- Multiple transaction_ids can legitimately share a receipt_number (multiple line items on one receipt), so this is flagged, not dropped.
CREATE OR REPLACE TEMP VIEW step7_flag_receipt_dupes AS
SELECT
  *,
  COUNT(*) OVER (PARTITION BY receipt_number) AS receipt_number_occurrences
FROM step6_fill_brand;
```

If the same transaction contains different SKUs, those rows may represent legitimate product lines. If there are confirmed duplicate transaction lines, a deduplication step can be applied. Do not automatically deduplicate using `transaction_id` alone without confirming what the ID represents.


Check for exact duplicates:

```sql
-- Check for exact duplicate transaction records.
-- A duplicate is defined as having the same customer, transaction, receipt, SKU, quantity, and total unit price.
SELECT COUNT(*) AS true_duplicate_rows
FROM (
  SELECT *, COUNT(*) OVER (PARTITION BY customer_id, transaction_id, receipt_number, product_sku_clean, quantity, total_unit_price) AS exact_dupe_count
  FROM step7_flag_receipt_dupes
) t
WHERE exact_dupe_count > 1;
-- Result: 0 exact duplicates found.
```

## Step 8: Calculate Unit Price and SKU Price Statistics

Calculate unit price:

```sql
-- STEP 8: Compute per-row unit price and per-SKU price variability
-- Derives unit_price from total_unit_price / quantity, then computes the standard deviation and median unit_price for each product_sku across the whole dataset. 
-- A high stddev on a given SKU signals inconsistent pricing -- possibly a data entry error, a promo mixed in with regular prices or the SKU code being reused for genuinely different products.
CREATE OR REPLACE TEMP VIEW step8_price_data AS
SELECT
  *,
  TRY_DIVIDE(total_unit_price, quantity) AS unit_price
FROM step7_flag_receipt_dupes;
```
## Step 9: Calculate sku_stats

Calculate median and standard deviation per SKU:

```sql
CREATE OR REPLACE TEMP VIEW step9_sku_stats AS
SELECT
  product_sku_clean,
  ROUND(STDDEV(unit_price), 2)                    AS stddev_unit_price,
  ROUND(PERCENTILE_APPROX(unit_price, 0.5), 2)     AS median_unit_price
FROM step8_price_data
GROUP BY product_sku_clean;
```

```sql
-- Preview: SKUs with the most inconsistent pricing
SELECT *
FROM step9_sku_stats
ORDER BY stddev_unit_price DESC
LIMIT 20;
```

## Step 10: Flag Price Outliers

Flag transactions where the total price is more than five times the expected median price for the quantity.

```sql
-- STEP 10: Flag individual rows as price outliers
-- A row is flagged when its total_unit_price is more than 5x what the SKU's median unit price would predict for that quantity. This is a row-level flag (not a drop) -- review flagged rows before deciding whether to correct or remove them, since some may be legitimate, for example, bulk/multi-pack pricing rather than errors.
CREATE OR REPLACE TEMP VIEW step10_price_outliers AS
SELECT
  p.*,
  s.stddev_unit_price,
  s.median_unit_price,
  ROUND(p.quantity * s.median_unit_price, 2) AS median_times_quantity,
  CASE
    WHEN p.total_unit_price > (5 * s.median_unit_price * p.quantity) THEN 'Yes'
    ELSE 'No'
  END AS flag_price
FROM step8_price_data p
LEFT JOIN step9_sku_stats s
  ON p.product_sku_clean = s.product_sku_clean;
```

Check the number of flagged rows: 

```sql
-- How many rows are flagged as price outliers?
SELECT COUNT(*) AS flagged_price_outlier_rows
FROM step10_price_outliers
WHERE flag_price = 'Yes';
```

Inspect them:

```sql
-- the worst offenders >:)
SELECT
  transaction_id, product_sku_clean, total_unit_price, quantity,
  unit_price, median_unit_price, stddev_unit_price, flag_price
FROM step10_price_outliers
WHERE flag_price = 'Yes'
ORDER BY stddev_unit_price DESC;
```

Price outliers are flagged rather than automatically deleted because some may represent legitimate bulk purchases, promotions, or other pricing situations.

## Step 11: Create the Silver Transactions Table

The final table is explicitly created in the Silver schema.

```sql
-- STEP 11: Build the final cleaned table
-- Rename cleaned columns back to their original names and drop helper/flag columns that were only needed for QA during cleaning.
-- Price outlier fields are kept as flags, not used to drop rows -- decide row removal only after reviewing Step 10's output.
CREATE OR REPLACE TABLE ftw.`03_silver`.retail_transactions_cleaned AS
SELECT
  customer_id,
  transaction_id,
  receipt_date_clean      AS receipt_date,
  transaction_date_clean  AS transaction_date,
  receipt_number,
  product_sku_clean       AS product_sku,
  product_brand_clean     AS product_brand,
  quantity,
  total_unit_price,
  unit_price,
  retailer,
  branch_clean            AS branch,
  receipt_number_occurrences,
  median_unit_price,
  stddev_unit_price,
  flag_price
FROM step10_price_outliers;
```

## Step 12: Final Transactions Validation

```sql
-- STEP 12: Final validation
-- Compare row counts before/after and re-check that each issue category is resolved (or intentionally flagged, not dropped).
SELECT
  (SELECT COUNT(*) FROM ftw.`02_bronze`.retail_transactions)   AS raw_row_count,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned AS rtc) AS clean_row_count,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE product_brand IS NULL) AS still_missing_brand,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE receipt_date IS NULL) AS still_missing_receipt_date,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE receipt_date > transaction_date) AS still_bad_date_order,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE quantity = 0 OR total_unit_price = 0) AS still_zero_values,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE flag_price = 'Yes') AS flagged_price_outliers,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned) AS total_rows,
  (SELECT COUNT(DISTINCT transaction_id) FROM ftw.`03_silver`.retail_transactions_cleaned) AS distinct_transaction_ids,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_transactions_cleaned WHERE receipt_number_occurrences > 1) AS rows_sharing_a_receipt_info_only;
```

---

# 2. Retail Customers

Source:

```text
ftw.`02_bronze`.retail_customers
```

Target:

```text
ftw.`03_silver`.retail_customers_cleaned
```

## Step 0: Profile Raw Data

Check row counts, missing values, and duplicate users.

```sql
-- Quick look at the raw row count before any cleaning
SELECT COUNT(*) AS raw_row_count
FROM ftw.`02_bronze`.retail_customers;
```

```sql
-- STEP 0 (diagnostic): Confirm schema/structure assumptions
-- Check for missing values, duplicate user_id, and fully duplicate rows
SELECT
  COUNT(*) 
    AS total_rows,
  SUM(CASE WHEN user_id IS NULL THEN 1 ELSE 0 END)          
    AS missing_user_id,
  SUM(CASE WHEN birthday IS NULL THEN 1 ELSE 0 END)         
    AS missing_birthday,
  SUM(CASE WHEN registered_date IS NULL THEN 1 ELSE 0 END)  
    AS missing_registered_date,
  COUNT(*) - COUNT(DISTINCT user_id)          
    AS duplicate_user_id_rows
FROM ftw.`02_bronze`.retail_customers;
```
```sql
-- Are there fully duplicate rows
SELECT COUNT(*) AS fully_duplicate_rows
FROM (
  SELECT *, COUNT(*) OVER (PARTITION BY user_id, birthday, registered_date) AS dup_count
  FROM ftw.`02_bronze`.retail_customers
) t
WHERE dup_count > 1;
```

## Step 1: Standardize Date Types


```sql
-- STEP 1: Parse birthday and registered_date into proper date types
-- Convert birthday and registered_date into consistent date/timestamp formats before further cleaning.
CREATE OR REPLACE TEMP VIEW step1_parsed_dates AS
SELECT *,
    CAST(birthday AS DATE) AS birthday_clean,
    CAST(registered_date AS TIMESTAMP) AS registered_date_clean
FROM ftw.`02_bronze`.retail_customers;
```

Validate:

```sql
-- How many rows failed to parse either date?
SELECT
  SUM(CASE WHEN birthday_clean IS NULL THEN 1 ELSE 0 END)         AS unparsed_birthdays,
  SUM(CASE WHEN registered_date_clean IS NULL THEN 1 ELSE 0 END)  AS unparsed_registered_dates
FROM step1_parsed_dates;
-- Result: 0
```

## Step 2: Check Customer Age

Calculate age at registration and flag implausible values.

```sql
-- STEP 2: Flag implausible birthdates
-- Age is computed relative to registered_date (when the account was actually created), not today's date, so age doesn't drift depending on when this script is re-run. 
-- Two implausible patterns exist in the raw data: birthdates implying an age over 100 (e.g. 10/06/1008, clearly a typo/century error) and birthdates implying an age under 13 (e.g. 1/17/2024, a newborn opening an account).
-- Flagged, not dropped -- review before deciding to exclude these rows.

CREATE OR REPLACE TEMP VIEW step2_flagged_age AS
SELECT
  *,
  ROUND(DATEDIFF(registered_date_clean, birthday_clean) / 365.25, 1) AS age_at_registration,
  CASE
    WHEN DATEDIFF(registered_date_clean, birthday_clean) / 365.25 > 100 THEN 'too_old'
    WHEN DATEDIFF(registered_date_clean, birthday_clean) / 365.25 < 13  THEN 'too_young'
    ELSE 'ok'
  END AS age_flag
FROM step1_parsed_dates;
```

Check the results:

```sql
-- How many rows fall into each flag category?
SELECT age_flag, COUNT(*) AS row_count
FROM step2_flagged_age
GROUP BY age_flag
ORDER BY row_count DESC;
```

Inspect flagged customers:

```sql
-- Inspect the flagged rows directly
SELECT user_id, birthday_clean, registered_date_clean, age_at_registration, age_flag
FROM step2_flagged_age
WHERE age_flag != 'ok'
ORDER BY age_at_registration DESC;
```

The age flags are retained for review rather than automatically deleting customers.

## Step 3: Check Date Order

A customer cannot register before their birthday.

```sql
-- STEP 3: Check date ordering
-- registered_date should never be before birthday_clean (can't create an account before being born). Included as a safety check even though no such rows were found during initial exploration.
SELECT COUNT(*) AS registered_before_birth
FROM step2_flagged_age
WHERE registered_date_clean < birthday_clean;
Result: 0
```

## Step 4: Create the Silver Customers Table

```sql
-- STEP 4: Build the final cleaned table
-- Rename cleaned columns back to their original names. age_flag is kept as an informational column rather than used to silently drop rows -- decide whether to exclude 'too_old'/'too_young' rows 

CREATE OR REPLACE TABLE ftw.`03_silver`.retail_customers_cleaned AS
SELECT
  user_id,
  birthday_clean         AS birthday,
  registered_date_clean  AS registered_date,
  age_at_registration,
  age_flag
FROM step2_flagged_age;
```

## Step 5: Final Customers Validation

```sql
-- STEP 5: Final validation
-- Compare row counts before/after and re-check that each issue category is resolved (or intentionally flagged, not dropped).

SELECT
  (SELECT COUNT(*) FROM ftw.`02_bronze`.retail_customers)          AS raw_row_count,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned)  AS clean_row_count,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned WHERE birthday IS NULL)         AS still_missing_birthday,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned WHERE registered_date IS NULL)  AS still_missing_registered_date,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned WHERE registered_date < birthday) AS still_bad_date_order,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned WHERE age_flag = 'too_old')      AS flagged_too_old,
  (SELECT COUNT(*) FROM ftw.`03_silver`.retail_customers_cleaned WHERE age_flag = 'too_young')    AS flagged_too_young,
  (SELECT COUNT(*) - COUNT(DISTINCT user_id) FROM ftw.`03_silver`.retail_customers_cleaned)        AS duplicate_user_id_rows;

```

---

# Key Approach

The cleaning process separates **correction, validation, flagging, and removal**.

Not every data-quality issue should result in deleting a row. Where possible, we correct the value or create a flag so the issue remains visible for review.

Temporary views are used as intermediate cleaning stages:

```text
Bronze
  ↓
Step 1
  ↓
Step 2
  ↓
Step 3
  ↓
...
  ↓
Final Silver Table
```

The Silver layer contains the cleaned and validated datasets that will be used for the Gold layer and dashboard.

```text
ftw
├── 02_bronze
│   ├── retail_transactions
│   └── retail_customers
│
├── 03_silver
│   ├── retail_transactions_clean
│   └── retail_customers_cleaned
│
└── 04_gold
    └── dashboard_data
```

The next stage is to join the cleaned customer and transaction tables in the Gold layer according to the business requirements.
