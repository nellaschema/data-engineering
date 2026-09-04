# TIL: Loading and Cleaning CSV Data with PySpark

Today I learned how to load a CSV into PySpark using an explicit schema and perform basic data-quality checks before cleaning the data.

## 1. Define an explicit schema

Instead of letting Spark infer the data types, I can define them myself:

```python
schema = StructType([
    StructField("InvoiceNo", StringType(), True),
    StructField("StockCode", StringType(), True),
    StructField("Description", StringType(), True),
    StructField("Quantity", IntegerType(), True),
    StructField("InvoiceDate", TimestampType(), True),
    StructField("UnitPrice", DoubleType(), True),
    StructField("CustomerID", DoubleType(), True),
    StructField("Country", StringType(), True),
])
```

Then load the CSV using that schema:

```python
df = (
    spark.read
    .format("csv")
    .option("header", "true")
    .schema(schema)
    .load(path)
)
```

Using an explicit schema gives me more control over how Spark interprets each column and avoids relying on schema inference.

## 2. Check data quality

I learned how to check for several common data-quality problems.

### Null values

```python
null_counts = df.select(
    [F.sum(F.col(c).isNull().cast("int")).alias(c) for c in df.columns]
)
```

This counts the number of null values in each column.

The important part is:

```python
F.col(c).isNull()
```

→ checks whether the value is null.

```python
.cast("int")
```

→ converts `True`/`False` into `1`/`0`.

```python
F.sum(...)
```

→ adds them up.

### Duplicate rows

```python
duplicates = df.count() - df.distinct().count()
```

The logic is:

```text
total rows - distinct rows = duplicate rows
```

### Invalid quantities

```python
invalid_qty = df.filter(
    F.col("Quantity") <= 0
).count()
```

This counts records where Quantity is zero or negative.

### Cancellation records

```python
cancellations = df.filter(
    F.col("InvoiceNo").startswith("C")
).count()
```

This identifies invoices whose `InvoiceNo` starts with `C`, which represents cancellation records in this dataset.

## 3. Clean the data

First, save the original row count:

```python
rows_before = df.count()
```

Then remove records without a CustomerID:

```python
df_clean = df.na.drop(subset=["CustomerID"])
```

`subset` tells `dropna()` which column to check.

So this means:

> Remove rows where `CustomerID` is null.

Fill remaining missing values:

```python
df_clean = df_clean.na.fill({
    "Description": "Unknown",
    "UnitPrice": 0.0
})
```

This replaces null descriptions with `"Unknown"` and null prices with `0.0`.

Remove duplicates based on the business key:

```python
df_clean = df_clean.dropDuplicates(
    subset=["InvoiceNo", "StockCode", "InvoiceDate"]
)
```

This keeps one record for each unique combination of InvoiceNo, StockCode, and InvoiceDate.

Finally, remove invalid quantities and cancellation records:

```python
df_clean = df_clean.filter(F.col("Quantity") > 0)

df_clean = df_clean.filter(
    ~F.col("InvoiceNo").startswith("C")
)
```

The `~` means **NOT**.

So:

```python
~F.col("InvoiceNo").startswith("C")
```

means:

> Keep invoices that do NOT start with `C`.

## 4. Compare before and after

```python
print(f"Rows before cleaning : {rows_before:,}")
print(f"Rows after cleaning  : {df_clean.count():,}")
```

This gives a simple validation of how many records were removed during cleaning.

The mental model:

```text
CSV
 ↓
Explicit schema
 ↓
Load DataFrame
 ↓
Check data quality
 ↓
Remove nulls
 ↓
Fill remaining nulls
 ↓
Remove duplicates
 ↓
Remove invalid quantities
 ↓
Remove cancellations
 ↓
Clean DataFrame
```

Key PySpark functions I learned:

```text
.schema()          → define column data types
.printSchema()     → inspect the schema
.isNull()          → check for null values
.distinct()        → get unique rows
.filter()          → keep rows matching a condition
.dropna()          → remove rows with nulls
.na.fill()         → replace null values
.dropDuplicates() → remove duplicate records
.startswith()      → check the beginning of a string
.count()           → count rows
```

The main lesson:

> **Before transforming data, first inspect its quality. Then apply cleaning rules based on the actual problems found.**
