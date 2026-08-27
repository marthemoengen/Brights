# Module 2: Silver Layer - Data Transformation

## 📖 Overview

In this module, you'll clean and transform the raw Bronze data into a validated Silver dataset. This layer applies data quality rules and creates a reliable foundation for analytics.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Apply data quality transformations
- Convert data types correctly
- Handle null values and data anomalies
- Create Delta Lake tables
- Understand the importance of data validation

---

## 📋 Prerequisites

Before starting this module:

1. ✅ Completed Module 1 (Bronze Layer)
2. ✅ CSV files exist in `Files/bronze/trips/`
3. ✅ You understand [Medallion Architecture](00-medallion-architecture.md)

---

## 🔧 Step-by-Step Instructions

### Step 1: Understand the Transformations

The Silver layer transforms raw data by:

| Transformation | Description | Why? |
|---------------|-------------|------|
| **Type Casting** | Convert strings to proper types | Enable calculations & comparisons |
| **Timestamp Parsing** | Parse datetime strings | Enable time-based analysis |
| **Null Handling** | Identify and flag nulls | Ensure data completeness |
| **Validation** | Apply business rules | Ensure data accuracy |
| **Lineage** | Preserve source file and processing date | Trace records back to Bronze |

### Step 2: Create the Silver Table

In Fabric Lakehouse, Silver data is stored as **Delta Lake tables** in the `Tables` folder.

The notebook will create: `Tables/silver_trips`

### Step 3: Run the Notebook

Open `notebooks/02-silver-transformation.ipynb` and run each cell.

---

## 📝 Data Quality Rules

### Rules Applied in Silver Layer

| Rule | Description | Action |
|------|-------------|--------|
| **Valid Timestamps** | started_at and ended_at must be valid | Set `is_valid` to false |
| **Positive Duration** | duration must be > 0 | Set `is_valid` to false |
| **Station IDs Present** | start/end station IDs required | Set `is_valid` to false |
| **Reasonable Duration** | Duration < 24 hours | Set `is_valid` to false |
| **Valid Coordinates** | Lat/Long within Oslo bounds | Document as an extension exercise |

The notebook keeps invalid records in `silver_trips` and marks them with `is_valid = false`; it does not delete or move them. This lets you reconcile every Silver row back to the Bronze input. If you later want separate accepted and rejected tables, add that as a deliberate design change rather than assuming it happened.

### Data Type Conversions

| Column | Bronze Type | Silver Type |
|--------|-------------|-------------|
| started_at | STRING | TIMESTAMP |
| ended_at | STRING | TIMESTAMP |
| duration | STRING | INT |
| start_station_id | STRING | STRING |
| start_station_latitude | STRING | DOUBLE |
| start_station_longitude | STRING | DOUBLE |
| end_station_id | STRING | STRING |
| end_station_latitude | STRING | DOUBLE |
| end_station_longitude | STRING | DOUBLE |

---

## 🏗️ Silver Table Schema

```sql
CREATE TABLE silver_trips (
    -- Primary identifiers
    trip_id             STRING,           -- Generated unique ID
    
    -- Timestamps
    started_at          TIMESTAMP,        -- Trip start time
    ended_at            TIMESTAMP,        -- Trip end time
    
    -- Duration
    duration_seconds    INT,              -- Trip duration in seconds
    
    -- Start station
    start_station_id    STRING,
    start_station_name  STRING,
    start_station_desc  STRING,
    start_latitude      DOUBLE,
    start_longitude     DOUBLE,
    
    -- End station
    end_station_id      STRING,
    end_station_name    STRING,
    end_station_desc    STRING,
    end_latitude        DOUBLE,
    end_longitude       DOUBLE,
    
    -- Metadata
    source_file         STRING,           -- Original file name
    ingestion_date      DATE,             -- When data was processed
    
    -- Data quality flags
    is_valid            BOOLEAN,          -- Passed all quality checks
    quality_issues      STRING            -- Description of any issues
)
USING DELTA
```

---

## ✅ Verification Checklist

After completing this module, verify:

| Check | How to Verify |
|-------|---------------|
| ✅ Delta table created | Check Tables/silver_trips in Lakehouse |
| ✅ Data types correct | Run DESCRIBE on the table |
| ✅ Row counts reconciled | Compare actual Silver totals to the Bronze input total |
| ✅ Quality flags populated | Query is_valid column |

---

## 📊 Expected Results

### Establish Your Baseline

After Silver finishes, calculate and record:

- Total rows in `silver_trips`
- Valid rows where `is_valid = true`
- Invalid rows where `is_valid = false`
- The count for each `quality_issues` value
- The difference between the Bronze input total and Silver total

For this notebook implementation, the Silver total should reconcile to the rows loaded from the successful Bronze files because invalid records are flagged rather than filtered out. Treat any difference as an investigation, not as a failure against a preset number.

---

## 🔍 SQL Queries You'll Use

### Preview Cleaned Data
```sql
SELECT * FROM silver_trips LIMIT 10;
```

### Check Data Quality Distribution
```sql
SELECT 
    is_valid,
    quality_issues,
    COUNT(*) as record_count
FROM silver_trips
GROUP BY is_valid, quality_issues
ORDER BY record_count DESC;
```

### Validate Timestamp Conversions
```sql
SELECT 
    started_at,
    ended_at,
    duration_seconds,
    TIMESTAMPDIFF(SECOND, started_at, ended_at) as calculated_duration
FROM silver_trips
LIMIT 5;
```

---

## ⚠️ Troubleshooting

### Common Issues

| Problem | Solution |
|---------|----------|
| "Table already exists" | Use `CREATE OR REPLACE TABLE` or drop first |
| "Cannot parse timestamp" | Check source date format |
| Memory errors | Process data in batches by month |
| Missing data | Verify Bronze files are complete |

---

## 🎓 Key Takeaways

1. **Silver = Validated Data** - Apply business rules before analysis
2. **Delta Lake Enables Time Travel** - You can query historical versions
3. **Schema Enforcement** - Define types explicitly for reliability
4. **Quality Tracking** - Flag issues rather than just deleting records

---

## 🏃‍♂️ Next Steps

Your data is now clean and validated. Next, you'll transform it into a dimensional model for analytics.

👉 **[Module 3: Gold Layer](03-gold-layer.md)** - Build dimensional model

---

## 📚 Additional Resources

- [Delta Lake Documentation](https://delta.io/)
- [Spark SQL Functions](https://spark.apache.org/docs/latest/sql-ref-functions.html)
- [Data Quality Best Practices](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-data-quality)
