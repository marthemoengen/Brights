# Advanced Modules — Oslo Bysykkel Data Modelling

This folder extends the core course (Modules 0–3) with four advanced topics. Each module builds on the Gold layer star schema and introduces a new dimensional modelling concept.

---

## 📁 Contents

| Module | Notebook | Documentation | Topic |
|--------|----------|---------------|-------|
| 4 | `notebooks/04-periodic-snapshot.ipynb` | `docs/04-periodic-snapshot.md` | Periodic Snapshot Fact Table |
| 5 | `notebooks/05-accumulating-snapshot.ipynb` | `docs/05-accumulating-snapshot.md` | Accumulating Snapshot Fact Table |
| 6 | `notebooks/06-dim-station-location.ipynb` | `docs/06-dim-station-location.md` | Station Location Dimension with Bydel |
| 7 | `notebooks/07-dim-weather.ipynb` | `docs/07-dim-weather.md` | Weather Dimension & Trip-Weather Correlation |

---

## 📋 Prerequisites

Before starting these modules, complete Modules 0–3 from the main course so that the following tables exist in your Lakehouse:

- `silver_trips`
- `dim_date`
- `dim_time`
- `dim_station`
- `fact_trips`

---

## 🏗️ What You Will Build

### Module 4 — Periodic Snapshot (`fact_daily_station_activity`)

A daily snapshot of activity at each station — how many bikes departed, how many arrived, and what the net flow was. Useful for the operations team to identify stations that need rebalancing.

**New concept:** Periodic snapshot tables capture state at a fixed interval. Rows are never updated; a new slice is appended for each period.

---

### Module 5 — Accumulating Snapshot (`fact_trip_lifecycle`)

Each trip is modelled as a process moving through three milestones:  
`Bike unlocked → Bike returned → Record processed`

The table tracks lag between milestones, duration buckets, and whether a trip is still "in-flight".

**New concept:** Accumulating snapshots have one row per process instance. Rows are *updated* as later milestones are reached (simulated with `MERGE INTO` in production).

---

### Module 6 — Station Location Dimension with Bydel (`dim_station_location`)

A new dimension table that enriches station GPS coordinates with Oslo **bydel** (borough) information using a bounding-box coordinate lookup. Enables geographic trip-flow analysis at the district level.

**New concept:** A dimension can model the same real-world entity at a different grain or with different attributes than an existing dimension. Each gets its own surrogate key.

---

### Module 7 — Weather Dimension & Correlation (`dim_weather`)

Introduces an entirely new data source: **daily weather for Oslo 2026** via the free Open-Meteo API (no API key required). `dim_weather` is a **conformed dimension** that shares `date_key` with `dim_date`, making it joinable to any fact table with a single extra `JOIN`.

**New concept:** Conformed dimensions use the same key across multiple fact tables, allowing different data sources to be analysed together without ETL changes.

---

## 🗺️ Extended Star Schema

After completing all four modules, your Lakehouse will contain:

```
                ┌───────────────┐
                │   dim_date    │◄──── dim_weather (conformed, same date_key)
                └──────┬────────┘
                       │
       ┌───────────────┼──────────────────┐
       │               │                  │
┌──────┴──────┐  ┌─────┴──────┐  ┌───────┴──────┐
│  fact_trips │  │fact_daily_ │  │ fact_trip_   │
│  (Module 3) │  │station_    │  │ lifecycle    │
│             │  │activity    │  │ (Module 5)   │
│             │  │(Module 4)  │  └──────────────┘
└──────┬──────┘  └────────────┘
       │
┌──────┴──────────────┐
│    dim_station       │◄──── dim_station_location (Module 6, via station_id)
│    dim_time          │
└─────────────────────┘
```

---

## 💡 Tips

- Run modules in order — each builds on the previous
- Read the doc file first to understand the concept before running the notebook
- Experiment with the analytical queries at the end of each notebook — try changing filters or adding new groupings
