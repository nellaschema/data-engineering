# August 15, 2026 Week 4 Journal

## Systems Thinking: Data Engineering

This session focused on understanding data engineering as a system rather than simply a collection of tools. A recurring theme was understanding where data comes from, how it changes throughout its lifecycle, and how to design systems that keep data reliable and usable.

## Data Lifecycle

We discussed the data lifecycle and how data can become "contaminated" at different points throughout the process.

Some possible sources of contamination include:

* Incorrect data entry
* Inconsistent formats
* Missing values
* Duplicate records
* Invalid values
* Errors introduced during transformation
* Issues during data transfer or integration

A key takeaway was that data quality issues can originate at different stages, so we need to understand the entire lifecycle rather than only looking at the final dataset.

---

## Data Formats

We were introduced to different ways of storing and exchanging data, including:

* CSV
* JSON
* Parquet

> "There are many file formats, and you need to learn which one to utilize for the job."

The important point is not simply knowing different formats, but understanding **when and why to use each one**.

---

## Column vs. Row Storage

We discussed the difference between row-oriented and column-oriented storage.

**OLTP (Online Transaction Processing)** is typically associated with operational systems where transactions are recorded and updated.

**OLAP (Online Analytical Processing)** is designed for querying and analyzing large amounts of data, such as in a data warehouse.

This connected to another recurring principle:

> **Use the right tool for the right problem.**

---

## Separation of Compute and Storage

We discussed the separation of **compute** and **storage**.

This allows compute resources to be scaled independently depending on the workload.

### Elasticity

Elastic infrastructure can expand or contract based on demand.

For example, a system may need more computing resources when processing a large dataset and fewer resources when the workload decreases.

---

## Big Data

We discussed the scale and complexity involved in big data systems.

> "If you really know what you're doing, people will throw money at you."

The discussion emphasized that understanding how to work with large-scale data and infrastructure is a valuable technical skill.

---

## Data Storage Architectures

We discussed the differences between:

* Database
* Data Warehouse
* Data Lake
* Data Lakehouse

The important distinction is understanding the purpose and characteristics of each architecture rather than treating them as interchangeable technologies.

---

# Databricks

We continued learning about Databricks and its role in building data solutions.

Key areas to explore:

* Creating jobs
* Creating pipelines
* Making data usable for business intelligence
* Completing Databricks certifications

The goal is to become comfortable enough with Databricks to build and operationalize data pipelines.

---

# Data Pipelines

One of the concepts that stood out was the idea of a three-stage pipeline.

> "Whenever I say pipeline, it's the three-stage pipeline. Raw, cleansed, and business-ready." — Sir Myk

Instead of thinking only in terms of Bronze, Silver, and Gold, we can think about the stages more conceptually:

**Raw → Cleansed → Business-ready**

The important idea is that data goes through controlled stages of transformation until it becomes usable for business purposes.

---

## Documentation

A major emphasis of the session was documentation.

> **"Document. Understand the data before transforming it."**

Before changing data, we should understand:

* Where the data came from
* What the columns represent
* What the values mean
* What problems exist
* What transformations are necessary
* Why each transformation is being performed

### Raw Data

> **ALWAYS KEEP A COPY OF THE RAW DATA.**

The original data should be preserved before any transformation takes place.

We also discussed **Delta** as the difference between the raw data and the altered/transformed data.

This makes it important to be able to explain not only *what* changed, but *why* it changed.

---

## Engineering Failure

> "Failure should happen for the right reasons. Engineer when failure should happen."

This highlighted that failures in a data pipeline should not simply be unexpected problems. Systems should be designed so that invalid or problematic data causes failures at appropriate points, rather than silently producing incorrect results.

---

# Group Activity

For each treatment/transformation stage, we were asked to document:

1. What did you do?
2. Why did you do it?
3. What was the basis for the transformation?
4. What changed in the data?
5. How did you validate the result?

The emphasis was on making every transformation **traceable and defensible**.

---

# Feedback From My Teammates

This week I also received some unexpected feedback about my contribution to the group.

Sir Mark approached me after the presentation and said:

> "Hello Nella, ang galing mo magpresent, naitawid mo yung output niyo kanina."

Ate Virna also told me:

> "Pinaguusapan ka pa lang namin Nella, ang galing mo magplan and very organized ka. Tapos sabi mo pa na feeling mo wala kang nacocontribute which is not true."

Sara added:

> "Oo nga tapos nagsorry pa siya na 'sorry guys mahilig ako sa table.' Like okay nga 'yon eh kasi ang organized."

Bri said:

> "Actually 'yon nga napansin ko sayo even before, like kapag maganda yung output alam ko ikaw gumawa eh."

### Reflection

This feedback was meaningful because I had been underestimating my contribution to the group.

I tend to think of contribution in terms of technical output or how much code I personally produced. The feedback made me realize that planning, organizing information, structuring the group's work, and presenting the final output are also meaningful contributions.

Apparently, my tendency to make tables is not a problem.

It might actually be one of my strengths.

---

# Key Takeaways

1. Understand the data before transforming it.
2. Always preserve the raw data.
3. Document every transformation and its basis.
4. Choose the appropriate file format and storage architecture for the problem.
5. Understand the difference between operational and analytical workloads.
6. Compute and storage can be scaled independently.
7. Data pipelines should move data from raw to cleansed to business-ready.
8. Systems should be designed to fail for the right reasons.
9. Data engineering is about designing reliable systems, not simply manipulating data.
10. Organization and communication are also valuable contributions to a technical team.
