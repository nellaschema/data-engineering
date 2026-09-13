# Week 8 - September 12, 2026 Journal

**Topic:** Data Ingestion, Incremental Processing, Data Volatility, APIs, and Web Scraping

## Table of Contents

- [What I Learned](#what-i-learned)
- [1. Batch vs. Streaming](#1-batch-vs-streaming)
- [2. Incremental Loading](#2-incremental-loading)
- [3. Append, Overwrite, and MERGE](#3-append-overwrite-and-merge)
- [4. Data Volatility and CDC](#4-data-volatility-and-cdc)
- [5. Idempotency and Deduplication](#5-idempotency-and-deduplication)
- [6. Late-Arriving Data](#6-late-arriving-data)
- [Data Sources](#data-sources)
  - [7. File Ingestion](#7-file-ingestion)
  - [8. Database Ingestion](#8-database-ingestion)
  - [9. API Ingestion](#9-api-ingestion)
  - [10. API Practice in Databricks](#10-api-practice-in-databricks)
  - [11. Data Provenance](#11-data-provenance)
  - [12. Schema Drift](#12-schema-drift)
  - [13. Web Scraping](#13-web-scraping)
  - [14. Web Scraping Is Fragile](#14-web-scraping-is-fragile)
  - [15. Understanding APIs Through the Browser](#15-understanding-apis-through-the-browser)
- [Key Mental Model](#key-mental-model)
- [Group Assignment: NYC Mobility Ingestion Challenge](#group-assignment-nyc-mobility-ingestion-challenge)
- [What I Want to Practice Next](#what-i-want-to-practice-next)
- [Key Takeaways](#key-takeaways)

## What I Learned

This week focused on a part of data engineering that I had previously thought of mostly as "getting data into a table." I learned that ingestion is more complicated because data is continuously changing, arriving at different times, and coming from different types of sources.

One statement from Sir Myk that stood out to me was:

> "Data engineers produce living, breathing systems."

The pipeline may work today, but the data will continue to arrive and change. Because of this, an ingestion pipeline needs to account for what is new, what has already been processed, what has changed, and what happens when something fails.

Another important takeaway was:

> "Design for failure, not just success."

Failure is not an unusual situation in a data pipeline. Missing files, bad records, network errors, partial loads, duplicate batches, late-arriving data, and schema changes are all possible states that the pipeline needs to handle.

---

## 1. Batch vs. Streaming

Data can generally arrive in two ways:

* **Batch:** Data is processed in groups.
* **Streaming:** Data is processed continuously.

The session mentioned that most data ingestion still occurs through batch processing, which helped me understand that streaming is not automatically the better solution. The choice depends on the requirements of the system.

> "Complexity has a cost." - Sir Myk

When designing a pipeline, we need to consider the available resources, including team size, time, and maintenance effort.

The question becomes:

> What can we trade off?

---

## 2. Incremental Loading

I learned that incremental loading means processing only data that is new or has changed instead of processing the entire source every time.

Metadata can help the pipeline determine what has already been processed.

Examples of metadata include:

* Timestamp
* Sequential ID
* Filename
* Partition/date
* Source change log

The basic idea is:

```text
Source
  ↓
What have I already processed?
  ↓
Process only new or changed data
```

---

## 3. Append, Overwrite, and MERGE

We discussed different ways of loading data:

| Method       | What it does                                     |
| ------------ | ------------------------------------------------ |
| `APPEND`     | Inserts new rows                                 |
| `OVERWRITE`  | Replaces existing rows                           |
| `MERGE INTO` | Updates existing records and inserts new records |

`MERGE INTO` is particularly useful when the incoming data can contain both new and changed records (**upsert**).

---

## 4. Data Volatility and CDC

Data does not always remain unchanged after it has been loaded.

We discussed:

### SCD Type 1

* One of the most common approaches.
* The old value is overwritten.
* Historical values are not retained.

### SCD Type 2

* Historical values are retained in the same table.
* Changes require some way to identify the different versions of a record, such as timestamps or old/new records.

We also discussed **Change Data Capture (CDC)**.

The distinction I learned was:

> CDC asks: "What changed in the source?"

> SCD asks: "How should we preserve that change?"

---

## 5. Idempotency and Deduplication

Another concept introduced this week was **idempotency**.

An idempotent pipeline produces the same intended result when the same operation is run multiple times.

In simple terms:

```text
Run pipeline
     ↓
Run it again
     ↓
Same intended result
```

Because pipelines may need to be rerun after failures.

`Deduplication` is also important because the same batch or records may be received more than once.

> "the pipeline ran successfully" does not necessarily mean the resulting data is `correct`.

---

## 6. Late-Arriving Data

We discussed a situation where data arrives later than expected.

For example:

```text
Expected:
May 28 data → May 28

Actual:
May 28 data → June 2
```

The pipeline therefore cannot assume that data always arrives in chronological order.

This is another reason why ingestion needs to track metadata and support incremental processing.

The key lesson was:

> "Design for failure, not just success."

Possible failure states include:

* Missing files
* Bad records
* Network errors
* Partial loads
* Duplicate batches
* Schema changes
* Late-arriving data

---

# Data Sources

Another major topic this week was the different ways data can enter a pipeline.

The four source patterns discussed were:

1. Files
2. Databases
3. APIs
4. Websites

A statement that stood out to me was:

> "Don't scrape what you can download."

The appropriate ingestion method depends on the source and what access is available.

---

## 7. File Ingestion

For file-based ingestion, an important question is:

> Which files have already been processed?

Folders can effectively become a data source when new files are continuously added.

This connects directly with incremental loading. Instead of processing every file every time, the pipeline needs a way to determine which files are new.

Metadata such as filenames, timestamps, and partitions can help with this.

---

## 8. Database Ingestion

For databases, we discussed the difference between full extraction and incremental extraction.

A full extraction repeatedly queries the entire database.

This can become expensive when the database contains terabytes of data and can also affect the source server's resources.

Instead, incremental extraction can use a field such as `updated_at`:

```sql
SELECT *
FROM orders
WHERE updated_at > last_processed;
```

The goal is to process only the records that need to be processed.

> "How do we stay efficient? Balanced form and function = elegance." - Sir Myk

---

# 9. API Ingestion

An API, or Application Programming Interface, provides a structured way for systems to communicate.

The basic flow I learned was:

```text
API Request
     ↓
API Response
     ↓
JSON
     ↓
Python
     ↓
DataFrame
     ↓
Bronze Table
```

An API request generally involves an endpoint and parameters.

Real APIs may also introduce additional challenges:

* Authentication
* Pagination
* Rate limits
* Request limits
* Errors
* Timeouts

When building an API pipeline, I also need to consider:

> Where am I, and what happens if one request fails?

---

## 10. API Practice in Databricks

I practiced calling a public weather API and loading the response into a Bronze table.

The exercise was to retrieve the previous seven days of weather data for BGC.

I used:

* `requests` to call the API
* `pandas` to create a DataFrame
* Spark to convert the Pandas DataFrame
* Delta format to write the Bronze table

The basic implementation was:

```python
import uuid
from datetime import datetime, timedelta, timezone

import pandas as pd
import requests
```

generate ingestion metadata:

```python
source_system = "open_meteo_historical_weather_api"
batch_id = str(uuid.uuid4())
ingested_at = datetime.now(timezone.utc)
```

define the 7-day window:

```python
end_date = datetime.now().date() - timedelta(days=1)
start_date = end_date - timedelta(days=6)
```

convert API response into a pandas df:

```python
data = response.json()
weather_df = pd.DataFrame(data["hourly"])
```

add metadata to the bronze data:

```python
weather_df["source_system"] = source_system
weather_df["ingested_at"] = ingested_at
weather_df["batch_id"] = batch_id
weather_df["location_name"] = "Bonifacio Global City, Taguig"
weather_df["latitude"] = data["latitude"]
weather_df["longitude"] = data["longitude"]
```

convert to Spark df and append it to a delta table:

```python
spark_df = spark.createDataFrame(weather_df)

(
    spark_df.write
    .format("delta")
    .mode("append")
    .saveAsTable("workspace.default.bronze_bgc_weather")
)
```

This was useful because it showed me how `an external API` can become another `data source` for a data engineering pipeline.

---

# 11. Data Provenance

One of the new concepts was **data provenance**.

For external data, we should capture metadata such as:

* `source_system`
* `source_file` or `source_url`
* `ingested_at`
* `batch_id`

The question behind this is:

> "Where did this row come from?"

The basic concept is:

```text
Data Record
     ↓
Metadata
     ↓
Bronze
```

This gives the data lineage information needed to trace records back to their source.

I also noticed that this connects with the metadata columns I have already been using in my previous ingestion work, such as `_source_file`, `_ingested_at`, and `_run_id`.

---

# 12. Schema Drift

Another problem discussed was **schema drift**, where the source changes without the pipeline knowing about it.

For example, a pipeline may expect:

```python
required = ["time", "temperature_2m"]

for column in required:
    assert column in weather.columns
```

If an expected column is missing, the assertion fails.

I initially did not fully understand why this was necessary. My understanding is that the purpose is to detect a source change before the pipeline silently produces incorrect or incomplete data.

The important principle was:

> "Failing loudly is better than silently producing bad data."

I still consider schema drift handling one of the more complicated topics from this session, but I understand the purpose better now: the pipeline should detect unexpected changes instead of assuming the source will always remain the same.

---

# 13. Web Scraping

We also discussed web scraping as another possible data source.

For HTML parsing, I learned about:

* BeautifulSoup
* `lxml`
* Python's built-in HTML parser
* Pandas `read_html()`

I was able to install `lxml` in Databricks and use Pandas to read an HTML table:

```python
import pandas as pd

tables = pd.read_html(
    "/Volumes/ftw/01_source_files/cloudflare/shared/week08/population.html"
)

cities = tables[0]
cities
```

For downloading the actual HTML document, I used `requests`:

```python
import requests

url = "https://cofibean.blogspot.com/"
response = requests.get(url)
html = response.text
```

use BeautifulSoup to parse the HTML:

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(response.text, "html.parser")
print(soup)
```

---

# 14. Web Scraping Is Fragile

One important point from the session was that web scraping can be fragile.

Potential problems include:

* Sources changing without warning
* Scrapers breaking
* Anti-scraping measures

Before collecting data from a website, we should consider:

```text
Can we access it?
Are we allowed?
Should we collect it?
```

This introduced an ethical consideration that I had not previously associated strongly with data ingestion.

> "Be careful, from an ethical point of view." - Sir Myk

---

# 15. Understanding APIs Through the Browser

I also learned that a website's frontend often communicates with its backend through APIs.

The browser's Developer Tools can help reveal these requests.

The Network tab can show information such as:

* Request headers
* Request payload
* Response headers
* API requests and responses

This means that when trying to understand how a website communicates with an API, the Network tab can provide clues about how the request is structured.

I found this particularly interesting because it connects what we see on a website with the underlying data flow.

---

# Key Mental Model

The biggest takeaway from this week is that ingestion has two sides.

### 1. Get the data

```text
Files
Databases
APIs
Websites
```

### 2. Handle the data

```text
Incremental Processing
Merge
Deduplicate
Validate
Rerun
```

A pipeline therefore needs to answer several questions:

```text
What is new?
What changed?
Have I processed it?
Is it valid?
Can I safely rerun?
Where did it come from?
What happens if something fails?
```

These questions apply regardless of whether the source is a CSV file, database, API, or website.

---

# Group Assignment: NYC Mobility Ingestion Challenge

The group assignment for this week is the **NYC Mobility Ingestion Challenge**.

The instruction is to:

> Build your own schema.

This will require applying the ingestion concepts discussed during the session, particularly source handling, incremental processing, metadata, and designing for failure.

---

# What I Want to Practice Next

For the next stage, I want to become more comfortable with:

* Incremental API ingestion
* Handling API failures
* Deduplication
* Idempotent pipelines
* `MERGE INTO`
* Schema drift detection
* Data provenance
* Designing ingestion pipelines around failure scenarios

I also want to better understand how these concepts translate into an actual production-style pipeline rather than treating ingestion as simply "getting data into a table."

---

# Key Takeaways

1. Data ingestion is not just about loading data. It is about managing data as it continuously arrives and changes.
2. Batch and streaming are different approaches, and the appropriate choice depends on requirements and complexity.
3. Incremental loading avoids repeatedly processing the entire source.
4. Metadata helps determine what has already been processed.
5. `APPEND`, `OVERWRITE`, and `MERGE INTO` represent different loading strategies.
6. CDC identifies what changed, while SCD determines how those changes are preserved.
7. Idempotency makes pipelines safer to rerun.
8. Late-arriving data and other failures should be expected when designing pipelines.
9. APIs introduce concerns such as pagination, authentication, rate limits, errors, and timeouts.
10. Bronze data should preserve the received data together with useful ingestion metadata.
11. Schema drift should be detected instead of silently producing incorrect data.
12. Web scraping can be fragile and should be evaluated from both technical and ethical perspectives.
13. Regardless of the source, the same engineering questions remain: what is new, what changed, what has been processed, whether it is valid, whether it can be safely rerun, and where it came from.
