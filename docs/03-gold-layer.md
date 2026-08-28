# Module 3: Gold Layer - Dimensional Model

## 📖 Overview

In this module, you'll transform the Silver data into a dimensional model (Star Schema) optimized for analytics and reporting. This is where data modelling skills shine!

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand dimensional modelling concepts
- Design a star schema from transactional data
- Create Fact and Dimension tables
- Implement surrogate keys
- Explain Slowly Changing Dimension (SCD) Type 1 and Type 2 patterns
- Optimize for analytical queries

---

## 📋 Prerequisites

Before starting this module:

1. ✅ Completed Module 2 (Silver Layer)
2. ✅ `silver_trips` table exists with valid data
3. ✅ You understand [Medallion Architecture](00-medallion-architecture.md)

---

## 🎓 Dimensional Modelling Concepts

### What is Dimensional Modelling?

Dimensional modelling is a design technique for data warehouses that makes data:
- **Easy to understand** for business users
- **Fast to query** for analytics
- **Flexible** for different analysis perspectives

### Key Components

#### 1. Fact Tables
- Store **measurements** (metrics, facts, events)
- Contain **foreign keys** to dimension tables
- Usually have many rows (millions/billions)
- Examples: FactTrips, FactSales, FactOrders

#### 2. Dimension Tables
- Store **descriptive attributes** (the "who, what, where, when")
- Have a **surrogate key** (artificial primary key)
- Usually have fewer rows (hundreds/thousands)
- Examples: DimDate, DimTime, DimStartStation, DimEndStation, DimCustomer

#### 3. Star Schema
- Central fact table connected to dimension tables
- Looks like a star when diagrammed
- Optimized for read-heavy analytical workloads

```
          ┌──────────────┐
          │   DimDate    │
          └──────┬───────┘
                 │
┌────────────┐   │   ┌────────────┐
│DimStart   │◄──┼──►│  DimTime   │
│Station    │   │   └────────────┘
└────────────┘   │
                 │
┌────────────┐   │
│DimEnd      │◄──┘
│Station     │
└────────────┘
                 │
          ┌──────┴───────┐
          │  FactTrips   │
          └──────────────┘
```

---

## 🏗️ Our Dimensional Model Design

### Star Schema for Oslo Bysykkel

We'll create the following tables in the separate `gold` schema:

| Table | Type | Description |
|-------|------|-------------|
| `gold.dim_date` | Dimension | Date attributes (year, month, day, etc.) |
| `gold.dim_time` | Dimension | Time of day attributes (hour, minute, period) |
| `gold.dim_start_station` | Type 1 dimension | Origin station details (name, location) |
| `gold.dim_end_station` | Type 1 dimension | Destination station details (name, location) |
| `gold.fact_trips` | Fact | Trip measurements (duration, counts) |

### Grain Definition

**The grain of FactTrips is: One row per bike trip**

This means each row represents a single trip from one station to another.

---

## 📊 Table Specifications

### DimDate

| Column | Type | Description |
|--------|------|-------------|
| date_key | INT | Surrogate key (YYYYMMDD format) |
| full_date | DATE | The actual date |
| year | INT | Year (2026) |
| quarter | INT | Quarter (1-4) |
| month | INT | Month number (1-12) |
| month_name | STRING | Month name (January, etc.) |
| week_of_year | INT | Week number (1-52) |
| day_of_month | INT | Day of month (1-31) |
| day_of_week | INT | Day of week (1=Monday, 7=Sunday) |
| day_name | STRING | Day name (Monday, etc.) |
| is_weekend | BOOLEAN | True if Saturday or Sunday |

### DimTime

| Column | Type | Description |
|--------|------|-------------|
| time_key | INT | Surrogate key (HHMM format) |
| hour | INT | Hour (0-23) |
| minute | INT | Minute (0-59) |
| time_of_day | STRING | Period (Morning, Afternoon, etc.) |
| is_rush_hour | BOOLEAN | True if 7-9 AM or 4-7 PM |
| hour_12 | INT | Hour in 12-hour format |
| am_pm | STRING | AM or PM |

### DimStartStation and DimEndStation

| Column | Type | Description |
|--------|------|-------------|
| station_key | INT | Role-specific surrogate key |
| station_id | STRING | Original station ID |
| station_name | STRING | Station name |
| station_description | STRING | Location description |
| latitude | DOUBLE | Latitude coordinate |
| longitude | DOUBLE | Longitude coordinate |

`gold.dim_start_station` contains stations used as trip origins, while `gold.dim_end_station` contains stations used as trip destinations. Each dimension has its own surrogate-key namespace.

### FactTrips

| Column | Type | Description |
|--------|------|-------------|
| trip_key | BIGINT | Surrogate key |
| date_key | INT | FK to DimDate |
| start_time_key | INT | FK to DimTime (start) |
| end_time_key | INT | FK to DimTime (end) |
| start_station_key | INT | FK to `gold.dim_start_station` |
| end_station_key | INT | FK to `gold.dim_end_station` |
| duration_seconds | INT | Trip duration in seconds |
| duration_minutes | DOUBLE | Trip duration in minutes |
| trip_count | INT | Always 1 (for summing) |

---

## 🔍 Understanding Surrogate Keys

### What are Surrogate Keys?

Surrogate keys are artificial keys (usually integers) that uniquely identify each row in a dimension table.

### Why Use Them?

| Reason | Explanation |
|--------|-------------|
| **Performance** | Integers are faster to join than strings |
| **Stability** | Business keys can change; surrogates don't |
| **History** | Enable tracking changes over time (SCD) |
| **Consistency** | Uniform key format across all dimensions |

### Example

```
Business Key:  "station_623"  →  Surrogate Key: 1
Business Key:  "station_412"  →  Surrogate Key: 2
```

---

## 🕰️ Slowly Changing Dimensions (SCD)

Dimensions change. A bike station can be renamed, moved, or assigned a new description while retaining the same business key (`station_id`). SCD patterns define how the warehouse records those changes.

| Pattern | Change handling | Best for |
|---------|-----------------|----------|
| **Type 1** | Replace the current attribute values. History is not retained. | Current-state reports where historical attributes are not needed. |
| **Type 2** | Create a new dimension row with a new surrogate key and validity dates. | Historical reporting where facts must use the attribute version that was valid at the event time. |

The core notebook creates `gold.dim_start_station` and `gold.dim_end_station` as **Type 1** dimensions. They represent the current station state separately for each trip role and keep the star-schema exercise simple.

The optional `gold.dim_station_scd2` extension demonstrates **Type 2** history with these columns:

| Column | Purpose |
|--------|---------|
| `station_scd_key` | Surrogate key for one historical station version |
| `station_id` | Stable business key from the source |
| `effective_from_date` | First date on which the version applies |
| `effective_to_date` | Final date on which the version applies |
| `is_current` | Indicates the active version |
| `version_number` | Sequence number for a station's versions |

For production, load a staged source snapshot incrementally and use `MERGE` logic to expire the old row and insert the new version. When building a fact table against a Type 2 dimension, join on both the business key and the event date, so each fact receives the surrogate key for the version valid at that time.

---

## ✅ Verification Checklist

After completing this module, verify:

| Check | How to Verify |
|-------|---------------|
| ✅ gold.dim_date created | Check Tables/ in Lakehouse |
| ✅ gold.dim_time created | Check Tables/ in Lakehouse |
| ✅ gold.dim_start_station created | Has all valid origin stations |
| ✅ gold.dim_end_station created | Has all valid destination stations |
| ✅ gold.dim_station_scd2 created (optional) | Has effective dates and one current version per station |
| ✅ gold.fact_trips created | All foreign keys populated |
| ✅ Relationships valid | Join queries return expected results |

---

## 📊 Expected Results

### Table Sizes

Do not compare your results with preset row counts. The source files can be partial or updated, and Silver quality rules determine how many records are valid. Record the actual row counts produced in your run for each dimension and the fact table.

Use the model summary query in the notebook to capture the results. Then explain:

- Why `gold.dim_time` has a predictable number of minute slots.
- Why `gold.dim_date` depends on the observed trip date range.
- Why the two station dimensions depend on the valid origin and destination stations present in the source files.
- Why `gold.fact_trips` depends on the valid Silver records and successful dimension joins.

### Sample Queries You Can Answer

After building the dimensional model, you can easily answer:

1. **How many trips per day of week?**
2. **Which stations are busiest during rush hour?**
3. **What's the average trip duration by month?**
4. **How does weekend usage compare to weekdays?**

---

## ⚠️ Troubleshooting

### Common Issues

| Problem | Solution |
|---------|----------|
| Missing dimension keys | Check that all Silver data has valid stations |
| Duplicate stations | Use DISTINCT when building dimension |
| Date key format errors | Ensure YYYYMMDD format consistently |
| Null foreign keys | Verify joins between fact and dimensions |

---

## 🎓 Key Takeaways

1. **Star Schema simplifies analysis** - Easy to understand and query
2. **Surrogate keys improve performance** - Integer joins are faster
3. **Dimensions provide context** - The "who, what, where, when"
4. **Facts are measurable** - Duration, counts, amounts
5. **SCD patterns preserve the right history** - Type 1 overwrites; Type 2 versions
6. **Grain matters** - Define what each fact row represents

---

## 🏃‍♂️ Next Steps

After completing this module, you can:
- Build Power BI reports on top of this model
- Add more dimensions (weather, events, etc.)
- Extend FactTrips to use the Type 2 station key through a date-range join
- Create aggregated fact tables for performance

---

## 📚 Additional Resources

- [Kimball Group: Dimensional Modelling Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)
- [Star Schema Design](https://en.wikipedia.org/wiki/Star_schema)
- [Microsoft: Data Warehouse Schema Design](https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/data-warehousing)
