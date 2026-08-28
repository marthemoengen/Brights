# Module 4 (Advanced): Periodic Snapshot Fact Table

## 📖 Overview

This module introduces the **periodic snapshot** fact table pattern — one of Ralph Kimball's three classic fact table types. You will build `fact_daily_station_activity`, which captures the state of each bike station at the end of every day.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Explain the difference between transaction, periodic snapshot, and accumulating snapshot fact tables
- Build a periodic snapshot with daily station-level metrics
- Calculate net bike flow to support operational rebalancing decisions
- Query trends over time using the snapshot

---

## 🎓 The Three Fact Table Patterns

| Pattern | Grain | Row behaviour | Best for |
|---------|-------|--------------|----------|
| **Transaction** | One row per event | Append-only | Detailed event log |
| **Periodic Snapshot** | One row per entity per period | New slice each period | Trends, inventory, flow |
| **Accumulating Snapshot** | One row per process instance | Updated as milestones hit | Pipeline / lifecycle |

---

## 🗂️ Table Design: `fact_daily_station_activity`

**Grain:** One row per station per calendar day.

| Column | Type | Description |
|--------|------|-------------|
| `snapshot_key` | BIGINT | Surrogate PK (date + station combined) |
| `date_key` | INT | FK → `dim_date` (YYYYMMDD) |
| `station_key` | INT | FK → `dim_station` |
| `trips_started` | INT | Bikes that left this station today |
| `trips_ended` | INT | Bikes that arrived at this station today |
| `net_bike_flow` | INT | `trips_ended - trips_started` (positive = bikes accumulating) |
| `total_duration_seconds` | BIGINT | Sum of all trip durations starting here |
| `avg_duration_seconds` | DOUBLE | Average duration of trips starting here |
| `snapshot_timestamp` | TIMESTAMP | End-of-day marker |

### Key Design Decisions

- **Grain is station-day** not trip — this is what makes it a snapshot, not a transaction table
- `net_bike_flow` is a derived measure directly useful for the operations team
- New rows are added each day; existing rows are **never updated**
- Because rows are immutable, the table can be partitioned by `date_key` for efficient time-range queries

---

## 🔗 Star Schema Diagram

```
         ┌─────────────────┐
         │    dim_date      │
         │─────────────────│
         │ date_key (PK)   │
         │ full_date       │
         │ is_weekend      │
         └────────┬────────┘
                  │
┌──────────────── ┼ ─────────────────────────┐
│  fact_daily_station_activity               │
│────────────────────────────────────────────│
│  snapshot_key                              │
│  date_key          (FK → dim_date)         │
│  station_key       (FK → dim_station)      │
│  trips_started                             │
│  trips_ended                               │
│  net_bike_flow                             │
│  total_duration_seconds                    │
│  avg_duration_seconds                      │
└──────────────── ┬ ─────────────────────────┘
                  │
         ┌────────┴────────┐
         │   dim_station    │
         │─────────────────│
         │ station_key (PK) │
         │ station_id       │
         │ station_name     │
         └─────────────────┘
```

---

## 💡 Analytical Use Cases

1. **Rebalancing operations** — identify stations where bikes consistently drain away (large negative `net_bike_flow`)
2. **Capacity planning** — find the busiest stations to prioritise maintenance
3. **Trend analysis** — how do daily departures change over weeks and months?
4. **Weekend vs weekday patterns** — join to `dim_date.is_weekend` for instant segmentation

---

## 📋 Prerequisites

1. ✅ Completed Module 3 (Gold Layer)
2. ✅ `silver_trips` table exists with valid data
3. ✅ `dim_date` and `dim_station` tables exist
