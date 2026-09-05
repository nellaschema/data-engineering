# Weekly Journal - September 05, 2026


## Table Topics

*Notes to be added.*

## Group Presentation

*Notes to be added.*

## Lecture - Sir Myk

### Documentation Matters

Sir Myk emphasized the importance of documentation:

> "You want other people to build on top of that. Overtime it becomes better to support collaboration."

The goal is not to create documentation for the sake of documentation. It should help other people understand, use, and build upon the work.

> "Focus on the core principles."

Communication is also an important engineering skill. The five-minute presentation format forces us to communicate the important parts without assuming that everyone has unlimited time.

> "What's important is you will be able to communicate important things. That's why 5 minutes lang because not a lot of people have all the time in the world."

---

## SQL vs Spark

Some key points from the discussion:

* Few companies actually use Spark directly.
* SQL is easier to get into but difficult to master.
* Spark is a tool, not a language like SQL.
* Spark becomes useful when we need to **execute something at scale**.
* Scale is largely about **data volume and distributed computation**.

[Apache Spark - Unified Engine for Large-Scale Data Analytics](https://spark.apache.org/?utm_source=chatgpt.com)

> "Nagpower ng globalization - software, internet. Customer nila buong mundo. So hindi na applicable yung for 1 worker lang yung Spark Engine."

The term **big data** is partly a marketing term, but the underlying problem is real: data and workloads can become too large for a single machine or traditional processing approach.

Spark is open source, while Databricks provides a managed platform built heavily around open-source technologies.

### When to Choose Which

**Snowflake** is a strong choice when the team prioritizes:

* Business intelligence
* Fast ad-hoc SQL querying
* Automated platform management
* Data sharing across business units

**Databricks** is a strong choice when workflows involve:

* Unstructured or semi-structured data
* Advanced machine learning
* Custom PySpark pipelines
* Collaborative notebook environments
* Large-scale distributed processing

> "As data engineers, if you know how to plan and optimize, then you can manage the cost."

---

## Scale

### Scale Formula

> **Scale = Data Volume × Distributed Computation**

Previously, scale was mostly associated with the amount of data.

Now, scale also involves the **cost and complexity of coordinating multiple machines to process that data**.

### Throughput

Throughput involves distributing data across networks and processing it efficiently.

When optimizing distributed workloads, we need to think about:

* Network speed
* CPU
* Memory
* Data movement
* Number of machines
* Workload distribution

The goal is to find a practical balance rather than simply throwing more resources at the problem. Humanity's favorite solution to every engineering problem is apparently "add more machines," until the bill arrives.

---

## SQL to Spark

A SQL-style transformation can be expressed using PySpark:

```python
from pyspark.sql import functions as F

orders = spark.table("`ftw-week-06`.`03-mart`.fact_order_products")

product_orders = (
    orders
    .groupBy("product_id")
    .agg(
        F.count("*").alias("order_count"),
        F.avg("reordered").alias("reorder_rate")
    )
    .orderBy(F.desc("order_count"))
)

product_orders.show(10)
```

The important point is that the transformation itself is not completely unfamiliar.

We already know how to:

* Filter data
* Group data
* Aggregate data
* Sort data
* Join tables
* Transform columns

The difference is learning how to perform these operations using Spark.

> "You already know how to transform data. You need to learn the difference when you're doing it with Spark. Ganyan when you're learning a new language."

### Maintainability

As engineers, the goal is not to write the most complicated or technically impressive code.

The goal is **maintainability**.

> "Perfect code nga wala naman makagamit."

Code that nobody else can understand or maintain is not particularly useful, regardless of how clever it is.

---

## Parallelism

**Parallelism** means multiple machines can work on different portions of the data at the same time.

Instead of relying on one extremely powerful machine, distributed systems can combine multiple machines to process a workload.

This is one of the fundamental ideas behind Spark.

---

## My Skill Priority

Based on the discussion, my current learning priorities are:

1. **SQL and Data Modeling**

   * Foundational for data engineering.
   * Non-negotiable core skill.

2. **Core Python**

   * File handling
   * API ingestion
   * Data structures
   * Scripting

3. **Data Warehousing and Orchestration**

   * SQL-based transformations
   * dbt
   * Airflow
   * Pipeline scheduling

4. **PySpark**

   * Distributed computation
   * Partition strategy
   * Memory tuning
   * Join optimization

### Does Spark Always Run Faster Than SQL?

No.

Spark is a trade-off.

If the dataset is small enough, SQL can be faster and simpler.

Sir Myk pointed out that the Instacart dataset, with around **3 million rows**, is actually considered relatively small in the context of distributed data processing.

I was like: 
> Instacart has 3 million rows and that's considered small? Wow.

The practical lesson is that technology should be selected based on the problem, not because Spark sounds more impressive.

> "There are times na okay ang Spark. Sometimes you have to keep it simple. Madalas decided na for you yung technology na gagamitin mo, that's already decided for you."

### Key Principle

> **"Complexity must earn its place."**

Spark does not replace SQL.

Data engineers need to be practical about choosing tools.

---

## How Spark Works

Sir Myk explained the basic Spark execution process:

**Driver → Partitions → Workers/Executors → Shuffle**

As users, we do not necessarily need to manage every internal detail.

We submit a job, and Spark handles the distributed execution.

That abstraction is one of the useful aspects of Spark.

---

## What Does "Expensive" Mean?

In data engineering and database optimization, an **expensive operation** is one that consumes significant computational resources, including:

* Memory (RAM)
* CPU cycles
* Network bandwidth
* Execution time

### Example: Joining Before Filtering

<img width="1024" height="558" alt="image" src="https://github.com/user-attachments/assets/ce2152c5-b8eb-417d-bb7f-1302cfa69133" />


In **Pipeline A**, large raw datasets are joined before filtering.

This can create several bottlenecks:

1. **Excessive memory usage** because the system has to load and process millions of unfiltered rows during the join, increasing memory pressure and the risk of out-of-memory errors.
2. **Wasted processing power** as the CPU performs join operations on rows that may eventually be discarded by a later filter.
3. **Network bottlenecks** - distributed query engines such as Spark, Snowflake, and BigQuery may need to shuffle large amounts of data across cluster nodes.
4. **Higher financial cost** because loud platforms can charge based on runtime, compute resources, or data scanned. Processing unnecessary data therefore increases cost.

### Pipeline B applies filtering before the join, which is related to **predicate pushdown** and early data reduction.

The general idea is:

```text
Raw Data
   ↓
Filter unnecessary rows
   ↓
Select required columns
   ↓
JOIN
   ↓
Transform / Aggregate
```

Reducing the amount of data before the join makes the join _smaller, faster, and more efficient_.

---

## When to Use SQL vs Spark

### Use SQL when:

* The data fits comfortably within the warehouse or platform.
* The workload mainly consists of joins and aggregations.
* The transformations are naturally relational.
* Simplicity matters.
* Performance is already sufficient.

### Use Spark when:

* Data volume becomes large.
* Distributed computation is necessary.
* Processing involves complex transformations.
* Data is unstructured or semi-structured.
* Custom PySpark processing is appropriate.

The important principle is:

> **Use Spark when the problem actually requires distributed computation.**

<img width="1245" height="792" alt="image" src="https://github.com/user-attachments/assets/772cf3cf-1ea0-4824-a88d-a41e8cc0c241" />


---

# 15-Minute Tool Setup: VS Code

The session also introduced connecting a local development environment to Databricks.

### Workflow

```text
Local IDE
   ↓
VS Code
   ↓
Databricks Extension
   ↓
Connect Workspace
   ↓
Open Project
   ↓
Run / Sync Command
   ↓
Databricks Compute
```

### Setup Steps

1. Open VS Code.
2. Open Extensions with `Ctrl + Shift + X`.
3. Search for **Databricks**.
4. Install the official Databricks extension.
5. Connect the extension to the Databricks workspace.
6. Open the project.
7. Test the connection with a simple command.

[Databricks VS Code Extension Installation Guide](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/install?utm_source=chatgpt.com)

### My Setup Experience

I encountered an error while trying to set up the Python environment in VS Code, which eventually led me to install **uv**.

I was able to successfully set up the Python environment with help from the integrated AI in VS Code.

However, I was not able to figure out:

1. Whether it is possible to open a Databricks folder directly inside VS Code.
2. How to create and run a SQL query in VS Code using the Databricks SQL connection.

### Spark Challenge

We were challenged to try setting up Spark locally after the program.

Sir Myk's advice was to learn enough Spark to pass the certification exam for now.

The more important takeaway for me was being able to understand the difference between SQL and Spark and knowing when each makes sense.

> Spark doesn't replace SQL. Data engineers must be practical.

---

# Talk: A Simple Guide to Testing Without Running Out of Limits

**Speaker: Sir Donald**

The session focused on testing within Databricks resource limitations.

### Key Points

* Avoid using streaming in the free edition because real-time ingestion consumes significant resources.
* Databricks usage limits may not provide enough warning before reaching the compute cap.
* Avoid `SELECT *`.
* Use `LIMIT` when testing queries.
* Do not waste compute resources during development.
* Use the FTW Databricks account only when it is necessary.
* Use personal accounts for general testing and practice.

> "Worst time to find out you've maxed your Databricks is during presentation day."

This is especially relevant when working on a project that needs to be demonstrated later.

### Query Management

> "Be imaginative when it comes to Python packages and setup."

The broader lesson was to be resource-conscious and find practical ways to manage development and testing workloads.

---

# Data Quality with SQL

**Speaker: Sir Hans**

> "A pipeline can run successfully but still produce bad data."

This was one of the most important reminders from the session.

A successful pipeline execution does not automatically mean the resulting data is correct.

### What Is Data Quality?

> **Quality = Meeting Expectations**

Before checking data quality, we need to define what "good data" means.

The goal is to establish expectations before problems occur.

### Data Quality Framework

For each validation, define:

| Parameter       | Question                          | Example                    |
| --------------- | --------------------------------- | -------------------------- |
| **WHAT**        | What are we checking?             | `clean_orders`             |
| **EXPECTATION** | What is acceptable?               | `user_id` must not be null |
| **THRESHOLD**   | How much failure can we tolerate? | 100% completeness          |
| **SEVERITY**    | Is it a warning or failure?       | 0 failures                 |
| **OWNER**       | Who investigates?                 | Data Engineer              |

### Data Quality Results

A `dq_check_results` table can summarize the parameters and results of each data quality check.

A useful data quality system should capture:

* What was checked
* What was expected
* What actually happened
* Whether the check passed
* How many records failed
* How severe the failure is
* Who is responsible for investigating it

> "Quality is not just accuracy." - Sir Myk

The goal is to:

```text
Build
  ↓
Check
  ↓
Learn
  ↓
Improve
  ↓
Repeat
```

A data quality system should be **action-oriented**.

Define a criterion, use it to evaluate the data, and then determine what action should be taken when the data does not meet the expected standard.

---

# Observability

Data quality should not be treated as a one-time validation step.

We also need to monitor the data flowing through the pipeline.

Important observability dimensions include:

* **Freshness**
* **Volume**
* **Schema**
* **Changes**
* **Failures**
* **Trends**
* **Lineage**
* **Monitoring**

The goal is to make the pipeline's behavior visible and detect problems before they become bigger issues.

---

# Reflection

One of our tasks was to **make our data observable**.

We struggled to create a data quality monitoring dashboard because we did not clearly establish what the dashboard was supposed to communicate.

We had an idea of what we wanted it to look like, but we did not have a clear structure for consolidating all of our validation checks into a useful monitoring view.

This made me realize that building the checks is only one part of data quality.

We also need to think about:

1. What information matters?
2. Who will use the dashboard?
3. What decisions should it support?
4. How should failures be prioritized?
5. What should trigger a warning versus a failure?
6. How do individual validation checks become an overall view of pipeline health?

The biggest takeaway from today's session was that **engineering is not just about making things work**.

It is about making systems `understandable`, `maintainable`, `observable`, and `practical`.
