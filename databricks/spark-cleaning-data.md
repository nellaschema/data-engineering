from pyspark.sql.types import StructType, StructField, StringType, DoubleType, IntegerType, TimestampType
from pyspark.sql import functions as F

# Replace with your own path
path = "/Volumes/databricks_ws_f13bea0c_ad21_4908_8ede_890d11fbf27c/default/course_datasets/online_retail.csv"

Define and load schema

```sql
# Define schema with all 8 columns
schema = StructType([ 
    StructField("InvoiceNo",   StringType(),    True),
    StructField("StockCode",   StringType(),    True),
    StructField("Description", StringType(),    True),
    StructField("Quantity",    IntegerType(),   True),
    StructField("InvoiceDate", TimestampType(), True),
    StructField("UnitPrice",   DoubleType(),    True),
    StructField("CustomerID",  DoubleType(),    True),
    StructField("Country",     StringType(),    True),
])

# Load CSV with explicit schema
df = (
    spark.read
    .format("csv")
    .option("header", "true")
    .schema(schema)
    .load('/Volumes/databricks_ws_f13bea0c_ad21_4908_8ede_890d11fbf27c/default/course_datasets/online_retail.csv')
)

df.printSchema()
```


<img width="651" height="188" alt="image" src="https://github.com/user-attachments/assets/f18ff24c-fe56-49f6-87f0-c9891ad5c339" />


Check nulls, duplicates, negative rows, and cancellations

```sql
# Count nulls per column
null_counts = df.select(
    [F.sum(F.col(c).isNull().cast("int")).alias(c) for c in df.columns]
)
null_counts.show()

# Count duplicate rows
duplicates = df.count() - df.distinct().count()
print(f"Duplicate rows        : {duplicates:,}")

# Count rows with zero or negative Quantity
invalid_qty = df.filter(F.col("Quantity") <= 0).count()
print(f"Invalid quantity rows : {invalid_qty:,}")

# Count cancellation rows (InvoiceNo starts with 'C')
cancellations = df.filter(F.col("InvoiceNo").startswith("C")).count()
print(f"Cancellation rows     : {cancellations:,}")
```

<img width="614" height="180" alt="image" src="https://github.com/user-attachments/assets/8af6492c-57cc-4c96-bf7d-f5d471ae9001" />


Clean Data

```sql
rows_before = df.count()

# Remove rows where CustomerID is null
df_clean = df.na.drop(subset=["CustomerID"])

# Fill remaining nulls with safe defaults
df_clean = df_clean.na.fill({"Description": "Unknown", "UnitPrice": 0.0})

# Remove duplicates by business key
df_clean = df_clean.dropDuplicates(subset=["InvoiceNo", "StockCode", "InvoiceDate"])

# Filter out zero/negative Quantity and cancellation records
df_clean = df_clean.filter(F.col("Quantity") > 0)
df_clean = df_clean.filter(~F.col("InvoiceNo").startswith("C"))

print(f"Rows before cleaning : {rows_before:,}")
print(f"Rows after cleaning  : {df_clean.count():,}")
```

<img width="601" height="58" alt="image" src="https://github.com/user-attachments/assets/cf9872f7-7b52-4cf4-8874-cf37e05bb3f3" />
