# August 08, 2026 - Week 3 at FTW

## Public Speaking Part 2

This week continued the focus on public speaking and communication.

### Grit

> "Grit is the ability to persist for long periods of time to achieve a goal."

> "Practice makes progress."

We discussed **Grit: The Power of Passion and Perseverance** by Angela Duckworth.

We also learned about **deliberate practice**, which involves:

1. Setting a stretch goal
2. Giving full concentration and effort
3. Seeking immediate feedback
4. Reflecting and refining

### Activity

**Group Toastmasters Mini Session**

---

# Data Analysis

## Group Activity: Employee Stress and Productivity

### Business Problem

> **In what ways can the company support employees in reducing their stress levels to improve work productivity?**

The exercise involved approaching the problem from a data perspective, starting from the data requirements and moving toward recommendations.

### Data Domains

* Wellness Survey
* Employee Information
* HR Log-ins

### 1. Check Data Quality

Before analyzing the data, assess:

* Consistency
* Accuracy
* Completeness
* Auditability
* Validity
* Uniqueness
* Timeliness

### 2. Treat Data Quality Issues

Examples:

* Remove duplicates
* Fix inconsistencies such as date formats
* Check formulated/calculated columns

### 3. Prepare the Data for Analysis

Potential transformations included:

* Combine employee wellness survey data with HR log-in data
* Create monthly overtime totals
* Categorize stress levels based on survey scores
* Create relevant groups such as department, shift, and employment type
* Compare stress levels across employee demographics such as age, gender, address, and accounts handled

### 4. Analyze the Data

Possible analyses:

* Average overtime hours per employee by department
* Monthly overtime trends
* Percentage of employees rendering overtime by shift
* Average stress score by department and shift
* Relationship between absenteeism and stress levels
* Departments or shifts with consistently high stress levels and overtime hours

### 5. Interpret the Results

The analysis should ultimately help answer the original business problem.

Potential findings:

* Identify departments or shifts with the highest workload and stress
* Determine whether higher stress is associated with higher overtime and lower productivity
* Identify periods when stress or overtime increases
* Determine which employee groups may require targeted intervention

Potential recommendations:

* Redistribute workload
* Adjust staffing
* Implement wellness programs
* Provide additional breaks
* Increase compensation proportionally to overtime hours
* Establish metrics to determine whether interventions are working

### Measuring Whether Interventions Work

Possible metrics:

* Stress scores
* Overtime hours
* Absenteeism
* Productivity metrics
* Employee wellness indicators

Visualizations and KPIs can be used to monitor these measures over time.

---

# Analytical vs. Critical Thinking

One of the most important lessons this week was the distinction between **analytical thinking and critical thinking**.

Analysis helps us understand what the data shows.

Critical thinking requires us to evaluate whether the information and conclusions are actually reliable.

> "Be objective in answering. Check each result. Provide proof that it's wrong. Be critical in thinking."

> "Look beyond the surface."

> "You’re gonna represent FTW in the future."

### A Lesson I Almost Missed

During a discussion about AI-generated analysis, we were asked what we thought about the results produced by AI.

My initial answer was:

> "Doubtful because AI is an LLM."

The answer the instructor was looking for was essentially:

**Is it reliable? Verify the results and provide evidence.**

I realized afterward that I had focused on *who or what produced the answer* rather than explaining **how to evaluate whether the answer is correct**.

The better approach is not simply to distrust AI because it is an LLM. Instead:

1. Examine the result.
2. Check the underlying data.
3. Reproduce the calculation or analysis.
4. Look for evidence supporting the conclusion.
5. Challenge the result and look for counterexamples.
6. Determine whether the conclusion is actually justified.

This was a useful reminder that being critical does not mean automatically rejecting an answer. It means **verifying it**.

---

# Day 3: Scaling the Solutions

## Operationalization

We were introduced to the concept of **operationalization**: moving analytical solutions beyond an isolated analysis and into something that can be used repeatedly within an organization.

> "Pwede nyo yan lahat gawin (data analytics, science). Ang starting point lang natin ay data engineering."

This helped put data engineering into perspective. Data engineering is not necessarily the final destination. It can serve as the foundation that enables analytics, data science, and other data-related work to operate at scale.

### Career Exploration

Activities included:

* Explore the PSF Human Capital Development, Learning and Organization Development track
* Review a reference organizational structure for a BI & Analytics team
* Consider where data engineering fits within an organization's data ecosystem
* Explore AWS Cloud and potential cloud scholarships

### Portfolio

We were introduced to the idea of building a portfolio, but the detailed process will be taught around Week 13.

---

# What is Data Engineering?

A major theme was **scalability**.

Data engineering involves building systems that can reliably collect, process, transform, store, and make data available for downstream users and applications.

We discussed:

* Data engineering lifecycle
* Data engineering undercurrents
* Data Science Hierarchy of Needs
* Why organizations need data engineers
* Programming and problem formulation
* Scaling data solutions
* Data quality
* Data integration

> "There will always be something wrong with the data."

### Data Engineering and Data Science

A key point was that data engineering provides the foundation for downstream activities such as analytics and data science.

Organizations are particularly likely to need data engineers when they:

* Generate large amounts of data
* Have complex data systems
* Need reliable data pipelines
* Need to integrate data from multiple sources
* Require scalable data infrastructure

### Correlation and Causation

> **Correlation does not imply causation.**

A relationship between two variables does not, by itself, demonstrate that one variable causes the other.

---

# Data Engineering Concepts

## Over-engineering

We discussed the risk of over-engineering, particularly the tendency to pursue new or fashionable tools simply because they are new.

A useful question to ask is:

> **Does this technology solve the actual problem?**

The newest tool is not necessarily the best tool.

## DAMA Wheel

The **DAMA Wheel** was introduced as an important reference for data management and data engineering.

## National ID

A TIL from the session was learning about the role of the National ID in enabling data integration across multiple government agencies.

---

# Databricks

We were introduced to the Databricks platform.

### Goal

Learn enough Databricks to eventually build an **end-to-end data pipeline**.

### Group Assignment

For the next exercise:

* Clean the data in Databricks using SQL
* Perform the cleaning before joining the datasets
* One person consolidates the group's work
* Prepare Exercises 2 and 3 for the following week
* Prepare for the presentation

### Databricks Genie 

Databricks Genie was introduced during the session. I asked it for instructions on how to create the dashboard, but instead of telling me how to do it, it just made the dashboard itself.

---

# Learning Resources

Additional resources recommended by Sir Myk Ogbinar included:

* Benjamin Robojan
* Data with Zack
* *Fundamentals of Data Engineering* book
* Seattle Data Guy?
* JoshDev


---

# Key Takeaways

1. **Data analysis starts with a question, not a dataset.**
2. **Data quality needs to be checked before analysis.**
3. **Analytical thinking and critical thinking are different.**
4. **AI-generated analysis should be verified rather than blindly accepted or rejected.**
5. **Data engineering provides a foundation for analytics and data science.**
6. **Scalability is a central concern in data engineering.**
7. **There will always be something wrong with data, so data quality is an ongoing process.**
8. **The newest technology is not automatically the right technology.**
9. **Data engineers need to understand the problem before choosing the solution.**
10. **The goal is not simply to learn tools, but to become capable of building reliable, scalable data solutions.**

## Reflection

This week changed how I think about data engineering. I initially associated the field primarily with technical tools and programming, but the sessions emphasized that the work begins much earlier: understanding the problem, identifying the right data, validating its quality, and critically evaluating the results.

The discussion about AI-generated analysis was particularly memorable. My instinct was to question the reliability of the output because it came from an LLM, but the more useful response is to verify the result and provide evidence. That distinction is something I want to carry forward as I continue learning.

The introduction to Databricks also made the transition from theory to practice more tangible. The immediate goal is not to master every tool, but to learn enough to build a working end-to-end pipeline and understand why each step is necessary.
