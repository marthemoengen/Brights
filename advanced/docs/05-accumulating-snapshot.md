# Module 5 (Advanced): Accumulating Snapshot Fact Table

## 📖 Overview

This module introduces the **accumulating snapshot** fact table — the most sophisticated of the three Kimball fact table patterns. You will build `fact_trip_lifecycle`, which tracks each individual bike trip through its lifecycle milestones.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand when to use an accumulating snapshot instead of a transaction fact table
- Model a process pipeline with multiple milestone timestamps and FK columns
- Calculate lag between milestones to measure process efficiency
- Explain how `MERGE INTO` keeps accumulating snapshots up to date in production

---

## 🎓 What Makes an Accumulating Snapshot?

An accumulating snapshot models a **business process** that moves through a predictable sequence of stages (milestones). Examples:

- 🛒 Order fulfilment: placed → picked → shipped → delivered
- 🏥 Patient journey: admitted → diagnosed → treated → discharged
- 🚲 Bike trip: unlocked → returned → processed

The key characteristics:
1. **One row per process instance** — not one row per event
2. **Multiple date/time foreign keys** — one per milestone
3. **Rows are updated** when later milestones are reached
4. **Lag columns** measure elapsed time between milestones

---

## 🗂️ Table Design: `fact_trip_lifecycle`

**Grain:** One row per bike trip.

| Column | Type | Description |
|--------|------|-------------|
| `trip_key` | BIGINT | Surrogate PK |
| `start_date_key` | INT | FK → `dim_date` (Milestone 1) |
| `start_time_key` | INT | FK → `dim_time` (Milestone 1) |
| `end_date_key` | INT | FK → `dim_date` (Milestone 2) |
| `end_time_key` | INT | FK → `dim_time` (Milestone 2) |
| `start_station_key` | INT | FK → `dim_station` |
| `end_station_key` | INT | FK → `dim_station` |
| `milestone_1_started_at` | TIMESTAMP | When bike was unlocked |
| `milestone_2_ended_at` | TIMESTAMP | When bike was returned |
| `milestone_3_processed_at` | TIMESTAMP | When record was ingested |
| `m1_to_m2_seconds` | INT | Trip duration (lag between M1 and M2) |
| `m1_to_m2_minutes` | DOUBLE | Same, in minutes |
| `m2_to_m3_seconds` | INT | Processing lag (M2 → M3) |
| `duration_bucket` | STRING | Short / Medium / Long / Very Long |
| `is_overnight_trip` | BOOLEAN | Did the trip cross midnight? |
| `is_complete` | BOOLEAN | All milestones populated? |

### Key Design Decisions

- **Multiple date/time FKs** let analysts answer "how long was the trip at the time it started?" and "how much time passed before it was processed?" independently
- `is_complete = FALSE` identifies **in-flight** records — trips that started but haven't returned (possible in a real-time feed)
- In production, use `MERGE INTO fact_trip_lifecycle USING new_milestones ON trip_key = ...` to update rows as each milestone arrives

---

## 🔗 Star Schema Diagram

```
dim_date ◄──────────────────────────────────────► dim_date
(start_date_key)                               (end_date_key)
       │                                              │
       │           fact_trip_lifecycle               │
       │    ┌──────────────────────────────────┐     │
       └───►│  trip_key                        │◄────┘
            │  start_date_key  (FK→dim_date)   │
dim_time ──►│  start_time_key  (FK→dim_time)   │◄── dim_time
            │  end_date_key    (FK→dim_date)   │
            │  end_time_key    (FK→dim_time)   │
            │  start_station_key (FK→dim_stn)  │◄── dim_station
            │  end_station_key   (FK→dim_stn)  │◄── dim_station
            │  milestone_1_started_at          │
            │  milestone_2_ended_at            │
            │  milestone_3_processed_at        │
            │  m1_to_m2_seconds                │
            │  m2_to_m3_seconds                │
            │  duration_bucket                 │
            │  is_complete                     │
            └──────────────────────────────────┘
```

---

## 💡 Analytical Use Cases

1. **Duration analysis** — how long are most trips? (duration bucket distribution)
2. **Processing lag** — how quickly does the pipeline ingest completed trips?
3. **Overnight trips** — flag unusual trips that span midnight
4. **Route analysis** — which station-to-station pairs are most popular?
5. **Time-of-day patterns** — join start_time_key to dim_time.time_period

---

## 📋 Prerequisites

1. ✅ Completed Module 3 (Gold Layer)
2. ✅ `silver_trips` table exists with valid data
3. ✅ `dim_date`, `dim_time`, and `dim_station` tables exist
