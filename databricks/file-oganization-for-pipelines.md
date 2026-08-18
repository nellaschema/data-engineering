## Organizing data in Databricks involves two separate layers: **where the raw files are stored** and **where the processed data is stored**.
### 1. Volumes = raw files

A Databricks volume can be used to store source files such as CSVs.

Instead of putting every file into one folder, organize them by dataset:

```text
raw_files/
├── customers/
│   ├── customers_01.csv
│   └── customers_02.csv
├── transactions/
│   ├── transactions_01.csv
│   └── transactions_02.csv
└── products/
    ├── products_01.csv
    └── products_02.csv
```

This is useful for automation because the file path can help determine which ingestion or transformation logic should process a file.

### 2. Catalogs and schemas = processed data

A typical medallion architecture:

```text
Catalog: ftw_project
├── bronze
│   ├── customers
│   ├── transactions
│   └── products
├── silver
│   ├── customers
│   ├── transactions
│   └── products
└── gold
    ├── sales_summary
    └── customer_summary
```

The different datasets do not need separate Bronze, Silver, and Gold schemas. They can have different columns and still exist as separate tables within the same schema.

### 3. The overall flow is:

```text
Volume
  ↓
Raw CSV files
  ↓
Bronze tables
  ↓
Silver tables
  ↓
Gold tables
  ↓
Dashboard
```

For example:

```text
Volume/customers/*.csv
          ↓
    bronze.customers
          ↓
    silver.customers
          ↓
 gold.customer_summary
```

The transformation logic explicitly defines which files or tables it works with. Databricks does not automatically know that a particular CSV belongs to a particular transformation.

This is why organizing raw files into dataset-specific folders is important **when building an automated pipeline**.

### 4. Automation

A **Job*** can orchestrate the transformations:

```text
JOB
├── customers Bronze → Silver
├── transactions Bronze → Silver
├── products Bronze → Silver
└── Silver → Gold
```

The Job can then be scheduled or configured with an appropriate trigger.

### Key takeaway

The simplest mental model is:

> **Volume = where the raw files live**

> **Catalog/schema/table = where the processed data lives**

> **Notebook = what transformation happens**

> **Pipeline = the data-processing flow**

> **Job = what orchestrates and schedules the work**
