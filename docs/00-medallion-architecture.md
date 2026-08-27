# Module 0: Medallion Architecture

## 📖 Introduction

The **Medallion Architecture** is a data design pattern used to logically organize data in a lakehouse. It progressively improves data quality as it flows through different layers: **Bronze**, **Silver**, and **Gold**.

This architecture is a best practice for building reliable, scalable, and maintainable data platforms.

---

## 🥉 Bronze Layer (Raw Data)

### Purpose
The Bronze layer serves as the **landing zone** for raw data. Data is stored exactly as it was received from the source system, with minimal or no transformation.

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Data State** | Raw, unprocessed, exact copy of source |
| **Format** | Original format (CSV, JSON, Parquet, etc.) |
| **Schema** | Schema-on-read (flexible) |
| **Quality** | No quality enforcement |
| **Use Case** | Data lineage, auditing, reprocessing |

### Best Practices

1. **Preserve Original Data**: Never modify source data
2. **Add Metadata**: Include ingestion timestamp, source file name, batch ID
3. **Organize by Source**: Structure folders by data source and date
4. **Enable Reprocessing**: Keep data for potential reloads

### Example Structure
```
bronze/
├── trips/
│   ├── 2026/
│   │   ├── 01.csv      # January 2026
│   │   ├── 02.csv      # February 2026
│   │   └── ...
│   └── _metadata/
│       └── ingestion_log.csv
```

### What Goes in Bronze?
- ✅ Raw CSV files downloaded from source
- ✅ Original data with all columns
- ✅ Metadata about the ingestion
- ❌ Cleaned or transformed data
- ❌ Aggregated data

---

## 🥈 Silver Layer (Cleaned Data)

### Purpose
The Silver layer provides **cleaned, validated, and enriched** data. This is where data quality rules are applied and the data is made consumable for analytics.

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Data State** | Cleaned, validated, deduplicated |
| **Format** | Delta Lake tables |
| **Schema** | Schema-on-write (enforced) |
| **Quality** | Data quality rules applied |
| **Use Case** | Enterprise-wide analytics, data science |

### Best Practices

1. **Enforce Schema**: Use Delta Lake with defined schemas
2. **Apply Data Quality**: Validate data types, ranges, null values
3. **Standardize Formats**: Consistent date formats, naming conventions
4. **Track Changes**: Use Delta Lake's versioning capabilities
5. **Document Transformations**: Clear lineage from Bronze

### Data Quality Transformations

| Transformation | Description | Example |
|---------------|-------------|---------|
| **Type Casting** | Convert to correct data types | String → Timestamp |
| **Null Handling** | Deal with missing values | Replace, remove, or flag |
| **Deduplication** | Remove duplicate records | Based on natural key |
| **Standardization** | Consistent formats | Dates, names, codes |
| **Validation** | Enforce business rules | Duration > 0 |

### Example Structure
```
Tables/
├── silver_trips           # Cleaned trip data
└── silver_trips_rejected  # Records that failed quality checks
```

### What Goes in Silver?
- ✅ Cleaned, validated records
- ✅ Properly typed columns
- ✅ Deduplicated data
- ✅ Standardized formats
- ❌ Aggregated/summarized data
- ❌ Dimensional models

---

## 🥇 Gold Layer (Business-Ready Data)

### Purpose
The Gold layer contains **business-level aggregates and dimensional models**. Data is organized for specific business use cases and optimized for reporting and analytics.

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Data State** | Aggregated, modeled, business-ready |
| **Format** | Delta Lake tables (Star Schema) |
| **Schema** | Dimensional model (Facts & Dimensions) |
| **Quality** | Production-ready |
| **Use Case** | BI reports, dashboards, KPIs |

### Best Practices

1. **Design for Consumers**: Optimize for query patterns
2. **Use Dimensional Models**: Star schema or snowflake
3. **Pre-aggregate When Useful**: Create summary tables
4. **Maintain Referential Integrity**: Foreign key relationships
5. **Version Control**: Track model changes over time

### Dimensional Modeling Concepts

#### Fact Tables
- Contain **measurements** (metrics, facts)
- Reference dimension tables via **foreign keys**
- Are at the grain of individual business events
- Examples: FactTrips, FactSales, FactOrders

#### Dimension Tables
- Contain **descriptive attributes**
- Have a **surrogate key** (PK)
- Enable filtering, grouping, and labeling
- Examples: DimStation, DimDate, DimTime, DimCustomer

### Star Schema Example
```
          ┌──────────────┐
          │   DimDate    │
          └──────┬───────┘
                 │
┌──────────┐     │     ┌───────────┐
│DimStation│◄────┼────►│  DimTime  │
└──────────┘     │     └───────────┘
                 │
          ┌──────┴───────┐
          │  FactTrips   │
          └──────────────┘
```

### What Goes in Gold?
- ✅ Fact tables with business metrics
- ✅ Dimension tables with attributes
- ✅ Pre-computed aggregations
- ✅ Data marts for specific domains
- ❌ Raw or partially processed data
- ❌ Data with quality issues

---

## 🔄 Data Flow Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   SOURCE          BRONZE           SILVER            GOLD                │
│                                                                          │
│   ┌──────┐       ┌──────┐        ┌──────┐         ┌──────┐              │
│   │ CSV  │ ────► │ Raw  │ ─────► │Clean │ ──────► │ Star │              │
│   │Files │       │ Copy │        │Delta │         │Schema│              │
│   └──────┘       └──────┘        └──────┘         └──────┘              │
│                                                                          │
│   External       1:1 Copy        Quality          Business               │
│   System         As-Is           Rules            Model                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 Layer Comparison

| Aspect | Bronze | Silver | Gold |
|--------|--------|--------|------|
| **Purpose** | Preserve raw data | Clean & validate | Business ready |
| **Users** | Data engineers | Data analysts, scientists | Business users |
| **Query Pattern** | Rarely queried | Ad-hoc analysis | BI & reporting |
| **Update Frequency** | On ingestion | After bronze load | After silver load |
| **Data Model** | None | Normalized tables | Dimensional model |
| **Quality Level** | None | Validated | Production |

---

## 🎯 Why This Architecture?

### Benefits

1. **Reproducibility**: Raw data in Bronze enables reprocessing
2. **Auditability**: Full data lineage from source to report
3. **Flexibility**: Each layer serves different needs
4. **Quality**: Progressive data quality improvement
5. **Performance**: Gold layer optimized for queries
6. **Maintainability**: Clear separation of concerns

### When to Use

- ✅ Building enterprise data platforms
- ✅ Multiple data sources and consumers
- ✅ Need for data quality governance
- ✅ Complex transformation requirements
- ✅ Regulatory/audit requirements

---

## 🏃‍♂️ Next Steps

Now that you understand the Medallion Architecture, proceed to:

👉 **[Module 1: Bronze Layer](01-bronze-layer.md)** - Download and ingest raw data

---

## 📚 References

- [Databricks: Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- [Microsoft: Lakehouse Architecture](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview)
- [Delta Lake Documentation](https://delta.io/)
