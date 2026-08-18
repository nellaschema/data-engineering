
## Databricks uses the following hierarchy:

```text
Catalog
└── Schema
    ├── Tables
    ├── Views
    └── Volumes
```

A Volume is a Unity Catalog object that belongs to a schema.

## Organizing Raw Files

Use a Volume to store source files such as CSVs. For multiple datasets, organize the files into separate folders:

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
