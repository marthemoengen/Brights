# Dimensional Modelling Cheat Sheet

## Quick Reference for Star Schema Design

---

## 🎯 Core Concepts

### Fact Tables
| Concept | Description |
|---------|-------------|
| **Purpose** | Store measurable business events |
| **Contents** | Foreign keys + measures |
| **Grain** | One row per atomic business event |
| **Examples** | FactSales, FactTrips, FactOrders |

### Dimension Tables
| Concept | Description |
|---------|-------------|
| **Purpose** | Provide context for facts |
| **Contents** | Surrogate key + descriptive attributes |
| **Size** | Usually fewer rows than facts |
| **Examples** | DimDate, DimCustomer, DimProduct |

---

## 🔑 Key Types

| Key Type | Description | Example |
|----------|-------------|---------|
| **Surrogate Key** | Artificial integer key (PK) | `station_key = 42` |
| **Business Key** | Natural identifier from source | `station_id = "623"` |
| **Foreign Key** | Reference to dimension (FK) | `start_station_key → dim_station` |
| **Degenerate Dimension** | Business key in fact table | `order_number` in FactOrders |

---

## 📊 Common Dimensions

### Date Dimension
```sql
date_key        -- Surrogate key (YYYYMMDD)
full_date       -- The actual date
year, quarter, month, day
day_of_week, day_name
is_weekend, is_holiday
```

### Time Dimension
```sql
time_key        -- Surrogate key (HHMM)
hour, minute
time_of_day     -- Morning, Afternoon, etc.
is_rush_hour
```

### Location/Geography Dimension
```sql
location_key    -- Surrogate key
location_id     -- Business key
name, description
latitude, longitude
city, region, country
```

---

## 🏗️ Star Schema Pattern

```
           ┌─────────────┐
           │  DimDate    │
           └──────┬──────┘
                  │
┌─────────┐       │       ┌─────────┐
│DimProduct│◄─────┼──────►│DimStore │
└─────────┘       │       └─────────┘
                  │
           ┌──────┴──────┐
           │  FactSales  │
           └─────────────┘
```

---

## ✅ Design Checklist

### For Each Fact Table
- [ ] Define the grain (what does one row represent?)
- [ ] Identify all dimensions
- [ ] List all measures
- [ ] Determine foreign keys
- [ ] Consider degenerate dimensions

### For Each Dimension
- [ ] Create surrogate key
- [ ] Keep business key for reference
- [ ] Add all descriptive attributes
- [ ] Consider hierarchies
- [ ] Plan for slowly changing dimensions

---

## 📐 Grain Examples

| Business Area | Grain | One Row = |
|--------------|-------|-----------|
| Retail Sales | Transaction line | One product sold |
| Bike Sharing | Trip | One bike trip |
| Web Analytics | Page view | One page view |
| Call Center | Call | One customer call |
| Inventory | Daily snapshot | One product per day |

---

## 🔄 Common Transformations

### Bronze → Silver
```sql
-- Type casting
CAST(duration AS INT)
TO_TIMESTAMP(started_at)

-- Data quality
WHERE duration > 0
WHERE station_id IS NOT NULL
```

### Silver → Gold
```sql
-- Surrogate key generation
ROW_NUMBER() OVER (ORDER BY ...)

-- Date key format
CAST(DATE_FORMAT(date, 'yyyyMMdd') AS INT)

-- Time key format  
(HOUR(time) * 100 + MINUTE(time))
```

---

## 📈 Common Measures

| Measure Type | Examples | SQL |
|-------------|----------|-----|
| **Additive** | Sales, Quantity | `SUM(amount)` |
| **Semi-Additive** | Balance, Inventory | `SUM()` over some dims |
| **Non-Additive** | Ratios, Percentages | `AVG()` or calculate |

---

## 🎯 Quick SQL Patterns

### Joining Fact to Dimensions
```sql
SELECT 
    d.month_name,
    s.station_name,
    SUM(f.trip_count) as trips
FROM fact_trips f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_station s ON f.start_station_key = s.station_key
GROUP BY d.month_name, s.station_name
```

### Creating Dimension Table
```sql
CREATE TABLE dim_example AS
SELECT 
    ROW_NUMBER() OVER (ORDER BY id) as example_key,  -- Surrogate
    id as example_id,                                 -- Business key
    name,
    description,
    -- other attributes
FROM source_table
```

### Creating Fact Table
```sql
CREATE TABLE fact_example AS
SELECT 
    ROW_NUMBER() OVER (...) as fact_key,
    d.date_key,           -- FK to dim_date
    p.product_key,        -- FK to dim_product
    amount,               -- Measure
    quantity,             -- Measure
    1 as record_count     -- For counting
FROM source_table s
JOIN dim_date d ON ...
JOIN dim_product p ON ...
```

---

## 📚 Resources

- [Kimball Group Techniques](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/)
- [Star Schema Wikipedia](https://en.wikipedia.org/wiki/Star_schema)
- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
