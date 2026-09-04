# TIL: PySpark Joins, Execution Plans, and Aggregations

Today I learned how to combine a large DataFrame with a small lookup table, inspect Spark's execution plan, and aggregate the resulting data.

### 1. Create a calculated column

```python
df_revenue = df.withColumn(
    "Revenue",
    F.col("Quantity") * F.col("UnitPrice")
)
```

`withColumn()` creates a new column.

Here:

```text
Revenue = Quantity × UnitPrice
```

### 2. Broadcast join

```python
df_joined = df_revenue.join(
    F.broadcast(df_lookup),
    on="Country",
    how="left"
)
```

The large `df_revenue` DataFrame is joined with the smaller `df_lookup` table using `Country`.

`F.broadcast(df_lookup)` tells Spark that the lookup table is small enough to broadcast to the worker nodes. This can avoid an expensive shuffle and make the join more efficient.

`how="left"` means all rows from `df_revenue` are preserved, even when there is no matching country in the lookup table.

Mental model:

```text
Big DataFrame + Small Lookup Table
                ↓
       Broadcast small table
                ↓
          Join on key
```

### 3. Check the execution plan

```python
df_joined.explain(mode="formatted")
```

`explain()` shows how Spark plans to execute the DataFrame operations.

For a broadcast join, I can look for operators such as:

```text
BroadcastHashJoin
BroadcastExchange
```

This helps verify whether Spark is actually using the broadcast strategy.

Mental model:

```text
join()     → what I want to do
broadcast() → optimization strategy
explain()  → how Spark plans to execute it
```

### 4. Aggregate by region

```python
region_agg = (
    df_joined
    .groupBy("Region")
    .agg(
        F.round(F.sum("Revenue"), 2).alias("total_revenue"),
        F.count("Revenue").alias("order_count"),
    )
    .orderBy(F.col("total_revenue").desc())
)
```

`groupBy("Region")` groups the records by region.

`F.sum("Revenue")` calculates total revenue for each region.

`F.round(..., 2)` rounds the result to two decimal places.

`F.count("Revenue")` counts the non-null Revenue values for each region.

`alias()` gives the calculated columns meaningful names.

`orderBy(...desc())` sorts the results from highest to lowest total revenue.

The mental model:

```text
groupBy → split data into groups
agg     → calculate metrics for each group
orderBy → sort the results
show    → display the results
```

Overall, this workflow answers:

> **Which regions generate the most revenue, and how many records contribute to that revenue?**
