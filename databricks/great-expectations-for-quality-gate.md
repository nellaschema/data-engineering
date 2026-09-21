# How I Added Data Quality, CI/CD, and Deployment to a Databricks Project

Today I worked on our **FTW Week 08 NYC Mobility project**, mainly in **Databricks and GitHub**.

## Table of Contents

1. [Great Expectations](#part-1-great-expectations)
   * [What is Great Expectations?](#what-is-great-expectations)
2. [Reviewing the Existing Data Quality Setup](#part-2-reviewing-the-existing-data-quality-setup)
3. [Updating the Validation Notebook](#part-3-updating-the-validation-notebook)
4. [Connecting the Validation to the Pipeline](#part-4-connecting-the-validation-to-the-pipeline)
5. [Understanding the Quality Gate](#part-5-understanding-the-quality-gate)
6. [Reviewing the Databricks Job](#part-6-reviewing-the-databricks-job)
7. [Working with Databricks Asset Bundles](#part-7-working-with-databricks-asset-bundles)
8. [Setting Up the GitHub Actions Workflow](#part-8-setting-up-the-github-actions-workflow)
9. [Understanding CI vs CD](#part-9-understanding-ci-vs-cd)
10. [Keeping GitHub as the Source of Truth](#part-10-keeping-github-as-the-source-of-truth)
    * [Question I Had: Should I Delete the Notebook in Databricks?](#question-i-had-should-i-delete-the-notebook-in-databricks)
11. [What I Learned](#what-i-learned)


The work started with understanding and improving our data-quality validation, then connecting that validation to the Databricks job and finally setting up the CI/CD deployment workflow.

The overall goal was:

```text
Data Quality
     ↓
Pipeline Validation
     ↓
Quality Gate
     ↓
CI/CD
     ↓
Databricks Deployment
```

---

# Part 1: Great Expectations

## What is Great Expectations?

**Great Expectations (GX)** is a framework for validating data against predefined expectations.

Instead of manually checking whether our data looks correct, we define rules such as:

```text
Column exists
Column has the correct data type
Column does not contain unexpected nulls
Values are within an expected range
Keys are unique
References exist
```

The basic flow is:

```text
Data
 ↓
Expectations
 ↓
Validation
 ↓
Pass/Fail
```

We use GX because data-quality rules should be **explicit, repeatable, and automated**.

---

# Part 2: Reviewing the Existing Data Quality Setup

Before adding the quality gate, I reviewed the existing validation notebook and the checks we already had for the NYC Mobility pipeline.

Our pipeline follows the medallion architecture:

```text
Bronze
  ↓
Silver
  ↓
Gold
```

The validation covers the different layers and checks whether the outputs meet the rules we defined.

One important distinction I learned:

```text
Code runs successfully
        ≠
Data is valid
```

A Spark notebook can complete without an error _even when the resulting data has quality problems_.

That is why we need **explicit** data-quality checks.

---

# Part 3: Updating the Validation Notebook

## Where?

I worked in the **Databricks GX notebook** first for testing (See 02-great-expectations-quality-gate.ipynb)

## What did I change?

I updated the table references to explicitly identify the datasets being validated:

Example:
```python
bronze_green = spark.table("`ftw-week-08`.`01_bronze`.green_taxi")
silver_green = spark.table("`ftw-week-08`.`02_silver`.green_taxi")
gold_fact = spark.table("`ftw-week-08`.`03_gold`.fact_green_taxi_trip")

silver_zones = spark.table("`ftw-week-08`.`02_silver`.taxi_zones")
gold_zone = spark.table("`ftw-week-08`.`03_gold`.dim_taxi_zone")
etc.
```

Purpose: to make the validation notebook independent of whatever catalog or schema happens to be active. It also makes it immediately clear which Bronze, Silver, and Gold tables are being checked.

---

# Part 4: Connecting the Validation to the Pipeline

I then worked through where the validation belongs in the actual pipeline.

The flow is:

```text
Ingestion
   ↓
Bronze
   ↓
Silver
   ↓
Gold
   ↓
Validation
   ↓
Quality Gate
```

Validation needs to happen after the relevant data has been created. For example, we shouldn't validate the Gold fact table before the Gold task has completed (job dependencies).
---

# Part 5: Understanding the Quality Gate

Quality gate = final checkpoint for the pipeline.

Without one, a successful job might simply mean:

```text
All notebooks executed
```

With a quality gate:

```text
Pipeline executed
       ↓
Data-quality checks executed
       ↓
Checks passed
       ↓
Pipeline succeeds
```

**Purpose:** Prevent a technically successful pipeline run from being treated as a valid data run when important quality checks fail.

---

# Part 6: Reviewing the Databricks Job

## Where?

I worked with:

```text
resources/
└── nyc-mobility-job.yml
```

This file defines the Databricks job and its tasks.

---

# Part 7: Working with Databricks Asset Bundles

The project also uses:

```text
databricks.yml
```

together with the resources configuration.

The general structure is:

```text
databricks.yml
│
├── resources/
│   └── nyc-mobility-job.yml
│
└── notebooks/
```

The idea is to keep the Databricks project configuration in source control rather than manually configuring everything inside Databricks.

The deployment flow becomes:

```text
GitHub
   ↓
Databricks Asset Bundle
   ↓
Databricks
```

---

# Part 8: Setting Up the GitHub Actions Workflow

## Where?

I worked on:

```text
.github/
└── workflows/
    └── deploy-databricks.yml
```

The workflow deploys the Databricks project when relevant changes are pushed to `main`.

The trigger includes:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - databricks.yml
      - resources/**
      - notebooks/**
      - .github/workflows/deploy-databricks.yml

  workflow_dispatch:
```

## Why use `paths`?

It prevents unrelated changes in the repository from triggering a Databricks deployment.

`workflow_dispatch` allows the workflow to be triggered manually.

---

# Part 9: Understanding CI vs CD

Today I also clarified the difference between CI and CD.

### CI - Continuous Integration

CI checks changes to the code.

```text
Code change
    ↓
GitHub
    ↓
Automated checks
    ↓
Pass/Fail
```

### CD - Continuous Deployment

CD deploys the project.

```text
main
 ↓
GitHub Actions
 ↓
Databricks
```

Together:

```text
GitHub
   ↓
CI
   ↓
CD
   ↓
Databricks
```

---

# Part 10: Keeping GitHub as the Source of Truth

## Question I Had: Should I Delete the Notebook in Databricks?

While setting up CI/CD, I wondered:

> **“If my notebook is already in Databricks, should I delete it now that I'm deploying it through GitHub?”**

I learned that the important question is not whether the notebook exists in both places. It's **which one is the source of truth**.

The intended flow with Databricks Asset Bundles is:

```text
GitHub
  ↓
Asset Bundle
  ↓
Databricks
```

GitHub contains the source-controlled notebook while Databricks receives the deployed version that actually runs.

So before deleting a notebook manually in Databricks, I should first check whether it is already being managed by the Asset Bundle.

If it is, I should make changes in GitHub and let the deployment process update Databricks instead of manually maintaining two versions.

```text
GitHub = master document
Databricks = deployed copy
```

So: I shall edit the master copy and let the deployment process update the copy.

### Key takeaway

> **GitHub is the source; Databricks is the deployed environment. First verify that the Asset Bundle is managing the notebook before manually deleting or editing it.**

---

# What I Learned

The biggest shift in my mental model today was:

Before:

```text
Write notebook
    ↓
Run notebook
    ↓
Check result
```

After:

```text
Source Control
      ↓
CI Checks
      ↓
CD Deployment
      ↓
Databricks Job
      ↓
Pipeline Execution
      ↓
Data Quality Validation
      ↓
Quality Gate
```

A data pipeline isn't just about writing correct SQL or PySpark.

It also needs a way to:

* track changes
* test changes
* validate data
* manage dependencies
* deploy consistently
* detect failures

The goal is not simply:

> “The notebook runs/the dq checks pass.”

It is:

> “The pipeline can be validated, deployed consistently, and tell us when the resulting data does not meet our quality requirements.”
