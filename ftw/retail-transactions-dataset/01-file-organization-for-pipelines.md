
## Databricks uses the following hierarchy:

```text
Catalog
└── Schema
    ├── Tables
    ├── Views
    └── Volumes
```

A Volume is a Unity Catalog object that belongs to a schema. We can use this to store source files such as CSVs. For multiple datasets, we can organize the files into separate folders:

```text
ftw_project
└── raw
    └── source_files (Volume)
        ├── customers/
        ├── transactions/
        └── products/
```

## Medallion Architecture

Use separate schemas for the Bronze, Silver, and Gold layers:

```text
ftw_project
├── raw
│   └── source_files (Volume)
├── bronze
│   ├── customers
│   ├── transactions
│   └── products
├── silver
│   ├── customers
│   ├── transactions
│   └── products
└── gold
    └── dashboard_data
```

Different datasets can have different columns and still belong to the same schema. Each dataset is represented as its own table.

## Key Takeaway

> **Catalog** → top-level organization

> **Schema** → organizes tables, views, and volumes

> **Volume** → stores source files

> **Table** → stores structured data

> **Notebook** → contains transformation logic

> **Pipeline** → defines the data-processing flow

> **Job** → orchestrates and schedules the work

The overall flow is:

```text
Raw files in Volume
        ↓
     Bronze
        ↓
     Silver
        ↓
      Gold
        ↓
    Dashboard
```

*We’re organizing the project this way so that the raw source files remain separate from the processed data, and each transformation layer has a clear purpose. This makes the workflow easier to maintain, trace, automate, and rerun when new or updated files are provided.*

*The structure also allows us to ingest different datasets independently in Bronze, combine and clean them in Silver, and create business-ready data for the dashboard in Gold.*
