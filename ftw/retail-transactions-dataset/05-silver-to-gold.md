## Goal
Build the Gold layer from the cleaned Silver tables and prepare datasets for reporting and dashboard use.

The Gold layer contains three tables:

```text
03_silver
├── retail_customers_cleaned
└── retail_transactions_cleaned
          ↓
04_gold
├── retail_customers_gold
├── retail_transactions_gold
└── customer_transactions_joined
```

The first two Gold tables preserve the natural grain of each dataset. The joined table is then created for reporting that require customer and transaction information together.

---

## Sources and Targets

### Sources

```text
ftw.`03_silver`.retail_customers_cleaned
ftw.`03_silver`.retail_transactions_cleaned
```

### Gold Tables

```text
ftw.`04_gold`.retail_customers_gold
ftw.`04_gold`.retail_transactions_gold
ftw.`04_gold`.customer_transactions_joined
```

---

# Step 0: Integrity check

Before joining the datasets, check whether every `customer_id` in the transactions table exists as a `user_id` in the customers table.

```sql
SELECT COUNT(*) AS transactions_with_unmatched_customer_id
FROM ftw.`03_silver`.retail_transactions_cleaned t
LEFT ANTI JOIN ftw.`03_silver`.retail_customers_cleaned c
  ON t.customer_id = c.user_id;
```

A result of `0` means every transaction has a matching customer.
Why is this important? because an unmatched customer could cause missing customer information after the join.

---

# Step 1: Build Gold Customers Table

The customer table remains at the **customer grain**, meaning **one row** represents **one customer**.

We also calculate `current_age` for reporting.

`age_at_registration` tells us the customer's age when they registered, while `current_age` is calculated using today's date.

```sql
CREATE OR REPLACE TABLE ftw.`04_gold`.retail_customers_gold AS
SELECT
  user_id,
  birthday,
  registered_date,
  age_at_registration,
  ROUND(
    DATEDIFF(CURRENT_DATE(), birthday) / 365.25,
    1
  ) AS current_age,
  age_flag
FROM ftw.`03_silver`.retail_customers_cleaned;
```

### Validate

```sql
SELECT
  COUNT(*) AS retail_customers_gold_row_count,
  COUNT(DISTINCT user_id) AS distinct_customers
FROM ftw.`04_gold`.retail_customers_gold;
```

Expected:

```text
total rows = distinct customers
```

that is, if `user_id` is _unique_.

---

# Step 2: Build Gold Transactions Table

The transactions table remains at the **transaction line-item grain** meaning one row represents one transaction line item.

```sql
CREATE OR REPLACE TABLE ftw.`04_gold`.retail_transactions_gold AS
SELECT
  transaction_id,
  customer_id,
  receipt_date,
  transaction_date,
  receipt_number,
  product_sku,
  product_brand,
  quantity,
  total_unit_price,
  unit_price,
  retailer,
  branch,
  receipt_line_item_count,
  median_unit_price,
  stddev_unit_price,
  flag_price
FROM ftw.`03_silver`.retail_transactions_cleaned;
```

### Validate

```sql
SELECT
  COUNT(*) AS retail_transactions_gold_row_count,
  COUNT(DISTINCT transaction_id) AS distinct_transaction_ids
FROM ftw.`04_gold`.retail_transactions_gold;
```

Note that the number of distinct `transaction_id` values does not necessarily have to equal the number of rows because _a transaction can contain multiple product line items_.

---

# Step 3: Build the Joined Gold Table

The joined table combines customer information with transaction information.

Its grain is still **one row per transaction line item**.

Customer information will therefore repeat across the customer's transaction rows. This is intentional because this table is designed for reporting and dashboard use.

```sql
-- STEP 3: Build customer_transactions_joined
-- One row per transaction line item (customer fields repeat across a customer's transactions) -- this is intentional for this table; use retail_customers_gold directly for customer-level analysis to avoid needing DISTINCT.

CREATE OR REPLACE TABLE ftw.`04_gold`.customer_transactions_joined AS
SELECT
  t.transaction_id,
  t.customer_id,
  t.receipt_date,
  t.transaction_date,
  t.receipt_number,
  t.product_sku,
  t.product_brand,
  t.quantity,
  t.total_unit_price,
  t.unit_price,
  t.retailer,
  t.branch,
  t.receipt_number_occurrences,
  t.median_unit_price,
  t.stddev_unit_price,
  t.flag_price,
  c.birthday,
  c.registered_date,
  c.age_at_registration,
  c.current_age,
  c.age_flag
FROM ftw.`04_gold`.retail_transactions_gold t
LEFT JOIN ftw.`04_gold`.retail_customers_gold c
  ON t.customer_id = c.user_id;
```

We use `LEFT JOIN` so that all transaction records are retained even if a customer match is missing.

---

# Step 4: Final Validation

The final validation checks whether the join changed the transaction row count and whether customer information is missing.

```sql
-- STEP 4: Final validation
-- Confirms the join didn't drop or duplicate any transaction rows and that customer-side fields came through cleanly.

SELECT
  (SELECT COUNT(*) FROM ftw.`04_gold`.retail_transactions_gold)              AS retail_transactions_gold_rows,
  (SELECT COUNT(*) FROM ftw.`04_gold`.customer_transactions_joined)   AS joined_rows,
  (SELECT COUNT(*) FROM ftw.`04_gold`.customer_transactions_joined WHERE birthday IS NULL) AS unmatched_customer_rows,
  (SELECT COUNT(DISTINCT user_id) FROM ftw.`04_gold`.retail_customers_gold)   AS distinct_customers,
  (SELECT COUNT(DISTINCT customer_id) FROM ftw.`04_gold`.customer_transactions_joined) AS distinct_customers_in_joined_table;
```

The key check is:

```text
retail_transactions_gold_rows
            =
joined_rows
```

If these counts are different, something happened during the join that needs investigation.

---

# What's the need for three gold tables??

We keep three tables instead of immediately creating one large flat table.

```text
retail_customers_gold
        ↓
  Customer-level analysis

retail_transactions_gold
        ↓
 Transaction-level analysis

customer_transactions_joined
        ↓
 Dashboard
```

Because this gives us flexibility:

* `retail_customers_gold` can be used for customer-level analysis.
* `retail_transactions_gold` can be used for transaction-level analysis.
* `customer_transactions_joined` can be used when both customer and transaction attributes are needed together.

---

# Final Medallion Architecture

```text
SOURCE CSVs
    ↓
VOLUME
    ↓
02_BRONZE
├── retail_customers
└── retail_transactions
    ↓
CLEANING + VALIDATION
    ↓
03_SILVER
├── retail_customers_cleaned
└── retail_transactions_cleaned
    ↓
JOIN + BUSINESS-READY DATA
    ↓
04_GOLD
├── retail_customers_gold
├── retail_transactions_gold
└── customer_transactions_joined
    ↓
DASHBOARD
```

## Key Takeaway

Bronze stores the raw structured data, Silver contains cleaned and validated datasets, and Gold contains datasets prepared for analysis and reporting.

The Gold layer does not simply mean "join everything together." We preserve the natural grain of the individual datasets and create a separate joined table when a denormalized structure is useful for reporting.

`denormalization` means intentionally adding redundant data or grouping related data into fewer tables to speed up query performance

`grain` means the level of detail that a single row in a table represents. i.e., A table with one row per customer has a customer-level grain.
