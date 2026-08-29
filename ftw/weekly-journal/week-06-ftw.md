# August 29, 2026 - Week 6 Journal

### Activity 1: Table Topics
- We did the session by group
- I got really distracted by the noise but Joy, one of my groupmates at the time, reassured me that it's okay and suggested we go outside the room so I can focus on my speech.

### Activity 2: Group Presentation
- Two of my groupmates presented our output and I felt so proud of us.

---

# Data Modeling

A major theme this week was the importance of having a consistent structure when working with data.

> "When building models, you want to have a consistent structure." — Sir Myk

This raised an interesting question:

> **What if you have 10,000 tables?**

The point was not simply to create more tables, but to think carefully about how data is structured, organized, and modeled at scale.

### Applying Data Modeling to Biochemical Genetics

One idea that came to mind was:

> **What if we created a star schema for biochemical genetics data?**

A possible starting structure could include entities such as:

* Patient
* Sample
* Requisition

This made the concepts of data modeling feel more relevant to my own domain.

---

# Meaningful Data

> "We work with meaning. By being able to connect those information we get meaningful insight."

Data modeling is not only about organizing tables. The relationships between data allow us to connect information and eventually derive meaningful insights.

This reinforced the importance of understanding what the data represents before designing the model.

---

# Invoice vs. Invoice Line

We discussed situations where it may be appropriate to aggregate data one level above the most granular level.

For example, depending on the analysis, we may work with:

* **Invoice**
* **Invoice Line**

An invoice represents a transaction, while invoice lines represent the individual items within that transaction.

Choosing between them depends on:

1. **Business requirements** — What level of detail does the business question require?
2. **Data availability** — What information is actually available?
3. **Compute resource limitations** — What level of processing is practical?

The most detailed data is not automatically the best data for every analysis.

---

# Facts and Dimensions

A useful distinction in dimensional modeling:

**Facts** → What happened

**Dimensions** → The context behind what happened

For example, a sales fact might tell us that a transaction occurred and its amount, while dimensions provide context such as:

* Who purchased
* What was purchased
* When it happened
* Where it happened

---

# OLTP vs. OLAP

### OLTP — Online Transaction Processing

Manages real-time, day-to-day operational transactions.

Examples include systems that handle:

* Purchases
* Orders
* Payments
* Customer transactions

### OLAP — Online Analytical Processing

Used to analyze large volumes of historical data for:

* Business intelligence
* Reporting
* Trend analysis
* Decision-making

This reinforced the distinction between systems designed to **run the business** and systems designed to **analyze the business**.

---

# Data Engineering

> **"Data engineers help reduce the complexity of data processing."**

A principle that stood out:

> **Engineer once, reuse many times.**

The goal of engineering is not to repeatedly perform the same manual data processing. Instead, we can build reusable processes and pipelines that perform the work consistently.

Sir Myk also mentioned that having plenty of test data is useful when developing and validating data solutions.

---

# Business Questions and Data Analysis

We continued practicing how to turn broad business themes into specific, measurable questions.

The three themes were:

* **Grow**
* **Optimize**
* **Protect**

The process involved moving from:

**Business Theme → Business Question → Measure → Dimension → Query → Result → Validation**

---

## Grow

Example questions:

* Which artists are usually co-purchased in one invoice and how can we leverage this?
* How can we increase the number of customers in high-tier countries that currently have only a small customer base?
* Which artists have higher customer reach?

One selected question was:

> **How can we increase the number of customers in high-tier countries that currently have only a small customer base?**

The analysis focused on:

**Total Spend per Customer by Country**

---

## Optimize

Potential questions:

* How can we optimize pricing in countries below the benchmark?
* How does business sales performance change per quarter?
* Which sales representatives handle the highest number of customers relative to their revenue?
* What months can we improve our sales performance?

One direction we explored was:

> **How can we retain sales during the highest-revenue periods and improve performance during the lowest-revenue periods?**

The analysis could examine:

* Peak and low months
* Genre
* Country
* City
* Line amount

---

## Protect

Potential questions:

* Are we experiencing a drop in repeat purchases?
* Which customers are at risk of churn based on their last purchase activity?
* Which employees perform best during low-selling months?

My selected question was:

> **In what ways can we address the drop in repeat purchases of customers?**

The proposed metric was **Repeat Purchase Rate (RPR)**:

**RPR = Number of customers with 2 or more purchases / Total number of customers**

The next step was to examine the monthly trend and assess.

---

# GitHub

We also completed a GitHub-related assignment.

The group checked:

* GitHub account creation
* Username
* Account creation date
* Repository setup
* Ability to clone the FTW DE journal repository

---

# Instacart Assignment

We were assigned the **Instacart Raw → Clean → Mart** workflow.

The goal is to build a data pipeline that moves data through different stages:

**Raw → Clean → Mart**

We also began organizing the work through a task tracker containing:

* Date
* Task
* Assignee
* Status
* Meeting ID

Initial tasks included:

* Create a new workspace
* Create a new GitHub repository
* Begin the Instacart workflow

---

# Next Week

### Present

1. Chinook Repository
2. Instacart Raw → Clean → Mart

### Not Required for Presentation

* Three new themes

---

# Body Literacy — Part 2

We continued the discussion on body literacy, including the different phases of the menstrual cycle and how they may be considered when planning activities, decision-making, and productivity.

---

# Key Takeaways

1. Data models need consistent structures, especially as the number of tables grows.
2. Data modeling should preserve the meaning and relationships within the data.
3. A star schema could potentially be applied to domain-specific datasets such as biochemical genetics data.
4. The appropriate level of data granularity depends on the business question.
5. Facts describe what happened, while dimensions provide context.
6. OLTP systems support operational transactions, while OLAP systems support analysis.
7. Data engineering reduces the complexity of repeated data processing.
8. A good engineering principle is: **Engineer once, reuse many times.**
9. Business questions need to be translated into measurable metrics and dimensions.
10. Query results should be validated rather than automatically accepted.
11. Raw → Clean → Mart provides a useful framework for thinking about data pipelines.

## Reflection

This week helped connect data modeling with the idea of meaning. It is easy to think of databases as simply collections of tables, but the real value comes from understanding how those tables relate to one another and what each piece of information represents.

I also found myself thinking about how these concepts could apply to biochemical genetics data. A model involving patients, samples, and requisitions could potentially make relationships within laboratory data much clearer and create a foundation for analysis.

The idea of **"Engineer once, reuse many times"** also stood out to me. It captures one of the reasons I am interested in data engineering: instead of repeatedly doing the same data preparation manually, build a system that can do it reliably and repeatedly.
