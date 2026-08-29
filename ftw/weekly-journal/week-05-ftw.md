# August 22, 2026 - Week 5 Journal

## DISC Personality Types

We discussed the different DISC personality types and identified how different working and communication styles can show up within a team.

The activity helped us think about how personality differences can affect collaboration, communication, and task delegation.

---

# Group Presentation: Data Pipeline

We presented our group's output for the data pipeline.

### Lessons From the Presentation

> "Pag nagmemeeting kasi tayo wala tayong agenda." — Sara, 2026

**Lesson 1:** Meetings should have an agenda.

Having a clear agenda helps the team understand what needs to be discussed and prevents meetings from becoming unfocused.

**Lesson 2:** Apparently, we should have a script and a timer.

For a 5-minute presentation, knowing who is presenting which section and how much time each person has is important.

**Lesson 3:** Rename cells.

Small organizational details matter when working collaboratively. Clear naming makes the work easier for everyone to understand.

> "Baka next time kailangan na rin natin ng accountability for each process ganun." — Sara, 2026

**Lesson 4:** Assign accountability for each process.

For group technical work, it should be clear who is responsible for each stage of the pipeline so that tasks do not fall through the cracks.

---

# Body Literacy

We were introduced to body literacy and discussed the menstrual cycle.

### Homework

Track three metrics daily for seven days:

* Energy
* Mental state
* Physiological symptoms

The goal is to observe patterns in these metrics over time.

---

# From Engineering Problems to Modeling Problems

We discussed the difference between looking at historical data and approaching a problem from an engineering perspective.

**Historical data** can help answer:

> What happened in the past?

This raised the question:

> **What makes it "engineering" rather than analysis?**

The distinction is not simply the presence of data. Engineering involves designing and building systems that reliably collect, transform, store, and deliver data for continued use.

---

# External Storage and Databricks

We learned how external storage can be connected to Databricks.

### Cloudflare R2

Cloudflare R2 was introduced as an example of external object storage.

The workflow involved:

**External Storage → Databricks → Volume → Tables**

We practiced:

* Connecting external storage to Databricks
* Creating a volume
* Working with the Chinook dataset
* Creating tables from the data

This helped connect the concepts of storage, ingestion, and data processing.

---

# Relational Database Keys

We discussed keys in relational databases and the importance of maintaining relationships between tables.

## Referential Integrity

Referential integrity is a database rule that ensures relationships between tables remain valid and accurate.

For example, if a table contains a foreign key referencing a customer, that customer should exist in the referenced table.

We also discussed how Excel does not enforce relational database constraints in the same way. A spreadsheet can allow data to be entered without enforcing the relationships that a relational database would normally protect.

---

# SQL Joins

We practiced joining relational tables using SQL.

The exercise reinforced the importance of understanding how tables are related before combining them.

---

# Normalization and Data Modeling

We were introduced to normalization as a foundation for data modeling.

### Why Not Just Use One Big Table?

A single large table may initially seem simpler, but it can create problems such as:

* Data redundancy
* Inconsistent values
* Update anomalies
* Difficult maintenance

### Why Normalize?

Normalization helps:

* Establish a single source of truth
* Reduce redundancy
* Avoid anomalies
* Make data easier to maintain
* Represent relationships between entities more clearly

> **"More tables is just a consequence of normalization."**

The goal of normalization is not to create as many tables as possible. Additional tables are a consequence of organizing data properly according to its relationships and dependencies.

### Transitive Dependencies

We also introduced the concept of **transitive dependencies**, which is important when determining whether a relational table has been properly normalized.

---

# Key Takeaways

1. Good teamwork requires structure, not just participation.
2. Meetings should have clear agendas and objectives.
3. Presentation roles and timing should be planned in advance.
4. Accountability should be assigned across different stages of a technical process.
5. Engineering involves building systems, not simply analyzing historical data.
6. External storage can be integrated with Databricks for data processing.
7. Relational databases use keys and constraints to maintain valid relationships.
8. SQL joins allow related data to be brought together for analysis.
9. Normalization reduces redundancy and prevents data anomalies.
10. More tables are not inherently better; they are often a consequence of properly modeling relationships.
