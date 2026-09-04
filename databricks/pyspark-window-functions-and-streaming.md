# TIL: Window Functions & Streaming in PySpark

I finally got to the part where Spark stops being just `groupBy()` and starts doing things with rows that actually happened before or after the current row.

The two big ideas here are:

1. **Window functions** = calculate across rows without deleting the individual rows.
2. **Streaming** = process new data as it arrives instead of rereading everything.

---

## 1. Why `groupBy()` isn't always enough

I already know:

```python
df.groupBy("Customer_ID").agg(
    F.sum("Amount").alias("total_spend")
)
```

This gives me something like:

```text
Customer 1 → 50,000
Customer 2 → 72,000
Customer 3 → 91,000
```

Useful, but Spark has now collapsed all the transactions into summaries.

What if I want to see:

```text
Customer 3
Transaction 1 → 5,000  → running total: 5,000
Transaction 2 → 13,500 → running total: 18,500
Transaction 3 → 4,000  → running total: 22,500
```

That's where a **window function** comes in.

### Mental shortcut

```text
groupBy()
→ "Give me one answer per group."

Window
→ "Calculate using the group, but DON'T delete my rows."
```

This distinction is probably the most important thing to remember.

---

# 2. Running Total

The window:

```python
window_spec = (
    Window
    .partitionBy("Customer_ID")
    .orderBy("Date")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)
```

Then:

```python
df = df.withColumn(
    "running_total",
    F.sum("Amount").over(window_spec)
)
```

The syntax looks terrifying until I translate each piece.

### `partitionBy()`

```python
.partitionBy("Customer_ID")
```

Means:

> "Do this separately for each customer."

So Customer 1 doesn't get mixed with Customer 2.

### `orderBy()`

```python
.orderBy("Date")
```

Means:

> "Put that customer's transactions in this order."

Here, it's chronological order.

### `rowsBetween()`

```python
.rowsBetween(
    Window.unboundedPreceding,
    Window.currentRow
)
```

Means:

> "Start at the very first transaction and keep going until the current row."

So:

```text
Row 1:
[1]

Row 2:
[1, 2]

Row 3:
[1, 2, 3]

Row 4:
[1, 2, 3, 4]
```

Then:

```python
F.sum("Amount").over(window_spec)
```

means:

> "Sum the Amount column using that window."

### The mental formula

```text
WHO?
partitionBy()

WHEN / WHAT ORDER?
orderBy()

WHICH ROWS?
rowsBetween()

WHAT CALCULATION?
sum().over()
```

So when I see:

```python
F.sum("Amount").over(window_spec)
```

I should think:

> "Sum Amount across the window I just defined."

---

# 3. Why the running total works

Suppose Customer 3 has:

| Date  | Amount |
| ----- | -----: |
| Jan 1 |  5,000 |
| Jan 5 | 13,500 |
| Jan 8 |  4,000 |

The window moves down the rows:

```text
Jan 1
→ 5,000

Jan 5
→ 5,000 + 13,500
→ 18,500

Jan 8
→ 5,000 + 13,500 + 4,000
→ 22,500
```

The original rows are still there.

That's the big difference from `groupBy()`.

---

# 4. Ranking Customers

For ranking, I first need the total revenue per customer:

```python
customer_revenue = (
    df.groupBy("Customer_ID")
      .agg(
          F.sum("Amount").alias("total_revenue")
      )
)
```

Now I have:

```text
Customer 1 → 50,000
Customer 2 → 72,000
Customer 3 → 91,000
Customer 4 → 99,000
```

Then:

```python
rank_window = Window.orderBy(
    F.desc("total_revenue")
)

customer_revenue = customer_revenue.withColumn(
    "rank",
    F.rank().over(rank_window)
)
```

Because I didn't use `partitionBy()`, Spark ranks **everyone together**.

```text
Customer 4 → 99,000 → Rank 1
Customer 3 → 91,000 → Rank 2
Customer 2 → 72,000 → Rank 3
Customer 1 → 50,000 → Rank 4
```

### Important difference

```text
Window.partitionBy(...)
→ rank separately inside groups

Window.orderBy(...)
→ rank everyone globally
```

---

# 5. Streaming

Now we're moving from:

> "Analyze the data I already have."

to:

> "Keep processing new data when it arrives."

Imagine the transaction folder looks like:

```text
transactions/
    day1.csv
    day2.csv
    day3.csv
```

Tomorrow:

```text
transactions/
    day1.csv
    day2.csv
    day3.csv
    day4.csv
```

I don't want Spark to unnecessarily process everything from scratch every time.

Streaming lets Spark watch the directory and process **new files incrementally**.

### Mental shortcut

```text
Batch
→ "Here's all my data. Process it."

Streaming
→ "Here's my data source. Keep watching it."
```

---

# 6. `read` vs `readStream`

Batch:

```python
df = (
    spark.read
         .schema(schema)
         .csv(path)
)
```

Streaming:

```python
stream_df = (
    spark.readStream
         .schema(streaming_schema)
         .csv(path)
)
```

The important word is:

```text
readStream
```

It tells Spark:

> "This isn't just a normal DataFrame. Keep watching this source for new data."

Also, streaming requires an **explicit schema**.

Which is convenient because I already learned that explicitly defining schemas is safer than letting Spark guess types from CSVs. Apparently Spark's guessing abilities are not something we should trust with our livelihoods.

---

# 7. What is a Micro-batch?

Spark Structured Streaming usually processes incoming data in small batches.

So instead of:

```text
5,000 rows
→ process all at once
```

it can work conceptually like:

```text
New files
    ↓
Micro-batch 1
    ↓
Process
    ↓
Micro-batch 2
    ↓
Process
    ↓
Micro-batch 3
    ↓
Process
```

The stream stays ready for whatever comes next.

---

# 8. Checkpoints

This was the part that initially felt abstract.

A checkpoint is basically Spark's **memory of streaming progress**.

It records information about what has already been processed.

Think:

```text
Checkpoint
=
"Spark's receipt showing what it already ate."
```

If the stream crashes:

```text
Stream
  ↓
Processes file 1
Processes file 2
Processes file 3
  ↓
CRASH
```

When it starts again:

```text
Checkpoint
  ↓
"Files 1, 2, and 3 were already processed."
  ↓
Continue from there
```

So checkpoints help prevent unnecessary reprocessing and duplicate output.

---

# 9. Writing the Stream

```python
query = (
    stream_df.writeStream
        .format("delta")
        .option(
            "checkpointLocation",
            checkpoint_path
        )
        .option(
            "path",
            output_path
        )
        .trigger(availableNow=True)
        .start()
)

query.awaitTermination()
```

There are several things happening here.

### `writeStream`

```python
.writeStream
```

Means:

> "Write this streaming DataFrame as a stream."

### `.format("delta")`

Write the results as Delta.

### `checkpointLocation`

```python
.option("checkpointLocation", checkpoint_path)
```

Tells Spark where to save its progress information.

### `path`

```python
.option("path", output_path)
```

Tells Spark where the output data goes.

### `availableNow=True`

This one is worth remembering:

```text
availableNow=True
→ process everything currently available
→ then stop
```

So it behaves nicely for demos and scheduled pipelines.

It's basically:

> "Process whatever has arrived so far, then we're done."

---

# 10. `awaitTermination()`

```python
query.awaitTermination()
```

This means:

> "Wait for the streaming query to finish."

Without it, the notebook can move on while the streaming query is still running.

With:

```python
.trigger(availableNow=True)
```

the query processes the available data and eventually stops.

---

# 11. Monitoring the Stream

Two useful things:

```python
query.status
```

and:

```python
query.lastProgress
```

### `query.status`

Tells me the current state.

For example:

```text
stopped
```

### `query.lastProgress`

Gives information about the most recent micro-batch.

Things like:

```text
number of input rows
processing rate
batch information
```

So:

```text
status
→ "What is the stream doing right now?"

lastProgress
→ "What happened in the most recent batch?"
```

---

# 12. Checkpoint Recovery

Suppose I processed:

```text
day1.csv
day2.csv
day3.csv
day4.csv
day5.csv
```

and the checkpoint knows that all five were processed.

If I restart using the **same checkpoint**:

```python
.option(
    "checkpointLocation",
    checkpoint_path
)
```

Spark checks its saved progress.

It sees:

```text
day1 → already processed
day2 → already processed
day3 → already processed
day4 → already processed
day5 → already processed
```

So:

```text
New rows processed = 0
```

No duplicate processing.

This is why the checkpoint location matters. If I randomly change the checkpoint path every time, Spark doesn't have the same history to work from.

---

# My Mental Cheat Sheet

## Window functions

```text
groupBy()
→ reduce rows

Window
→ keep rows
→ calculate across rows
```

For a window:

```text
partitionBy()
→ WHO?

orderBy()
→ IN WHAT ORDER?

rowsBetween()
→ WHICH ROWS?

sum().over()
→ WHAT CALCULATION?
```

The pattern:

```python
FUNCTION().over(window)
```

---

## Streaming

```text
readStream
→ read new data continuously

writeStream
→ write streaming results

checkpoint
→ remember what was processed

availableNow=True
→ process what's available, then stop

awaitTermination()
→ wait for the query to finish

query.status
→ current state

query.lastProgress
→ latest batch information
```

## The two ideas in one picture

```text
BATCH ANALYTICS

Data
 ↓
groupBy / Window
 ↓
Analysis


CONTINUOUS DATA

New files
 ↓
readStream
 ↓
Process incrementally
 ↓
writeStream
 ↓
Delta
 ↓
checkpoint remembers progress
```

### What I actually need to remember

I do **not** need to memorize every line of the code.

I need to recognize the patterns:

```text
Need a summary?
→ groupBy()

Need a calculation but want to keep every row?
→ Window

Need calculations per customer?
→ partitionBy()

Need chronological calculations?
→ orderBy()

Need new files processed automatically?
→ readStream

Need to remember what was already processed?
→ checkpoint

Need to process available files and stop?
→ availableNow=True
```
