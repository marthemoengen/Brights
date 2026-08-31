# Module 6 (Advanced): Star Schema with Station Location & Bydel

## 📖 Overview

This module enriches the station dimensions with **Oslo bydel** (borough/district) data by introducing `gold.dim_station_location` — a new dimension table with its own surrogate key and a `bydel` column derived from station coordinates.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand why a separate location dimension can be valuable alongside an existing station dimension
- Assign stations to Oslo bydeler using coordinate-based lookup
- Build `gold.dim_station_location` with a surrogate `location_key`
- Query cross-bydel trip flows and origin/destination patterns

---

## 🎓 Concept: Geographic Dimension Enrichment

The raw trip data contains GPS coordinates for each station but no administrative boundary information. Adding **bydel** (borough) enables a whole new class of analysis:

- Which bydel generates the most bike trips?
- Are there commuting patterns between bydeler (e.g., Grünerløkka → inner city)?
- Do inner-city stations behave differently from outer-area stations?

### Oslo Bydeler

Oslo is officially divided into **15 urban districts (bydeler)**:

| # | Bydel | Area type |
|---|-------|-----------|
| 1 | Gamle Oslo | Inner city |
| 2 | Grünerløkka | Inner city |
| 3 | Sagene | Inner city |
| 4 | St. Hanshaugen | Inner city |
| 5 | Frogner | Inner city |
| 6 | Ullern | Outer west |
| 7 | Vestre Aker | Outer west |
| 8 | Nordre Aker | Outer north |
| 9 | Bjerke | Outer east |
| 10 | Grorud | Outer east |
| 11 | Stovner | Outer east |
| 12 | Alna | Outer east |
| 13 | Østensjø | Outer south |
| 14 | Nordstrand | Outer south |
| 15 | Søndre Nordstrand | Outer south |

### Bydel Assignment Method

We use a **bounding-box coordinate lookup** (latitude/longitude ranges) embedded in `gold.ref_bydel`. This is a pedagogical simplification — in production you would use proper geometry functions (`ST_Within` with GeoJSON polygons, or H3 hexagonal indexing).

---

## 🗂️ Table Design

### `gold.ref_bydel` (reference / lookup)

| Column | Type | Description |
|--------|------|-------------|
| `bydel_number` | INT | Official Oslo district number (1–15) |
| `bydel_name` | STRING | District name in Norwegian |
| `lat_min / lat_max` | DOUBLE | Latitude bounding box |
| `lon_min / lon_max` | DOUBLE | Longitude bounding box |
| `area_type` | STRING | inner_city / outer_west / outer_east / outer_north / outer_south |

### `gold.dim_station_location`

**Grain:** One row per station (current state).

| Column | Type | Description |
|--------|------|-------------|
| `location_key` | INT | **Surrogate PK** (independent of `station_key`) |
| `station_id` | STRING | Business key — links to fact tables |
| `station_name` | STRING | Station display name |
| `station_description` | STRING | Street-level description |
| `latitude` | DOUBLE | GPS latitude (WGS84) |
| `longitude` | DOUBLE | GPS longitude (WGS84) |
| `bydel` | STRING | Oslo district name (or 'Unknown') |
| `bydel_number` | INT | Official district number |
| `area_type` | STRING | inner_city / outer_* |
| `is_inner_city` | BOOLEAN | Convenience flag |

### Why a New Surrogate Key?

`gold.dim_station_location` has its own `location_key` rather than reusing the role-specific keys from `gold.dim_start_station` or `gold.dim_end_station` because:
1. It represents a conformed location view across both station roles
2. It may change at a **different rate** — location/bydel classification is very stable, while station names or descriptions may change more often
3. It teaches the principle that different dimensions model **different aspects** of the same real-world entity

---

## 🔗 Star Schema Diagram

```
                ┌────────────────────────┐
                │ gold.dim_station_location│
                │────────────────────────│
                │ location_key (PK)      │
                │ station_id             │
                │ station_name           │
                │ latitude / longitude   │
                │ bydel                  │
                │ bydel_number           │
                │ area_type              │
                │ is_inner_city          │
                └──────────┬─────────────┘
                           │  (join on station_id)
                ┌──────────┴─────────────┐
                │       silver_trips      │
                │  (or fact_trips)        │
                └─────────────────────────┘
```

---

## 💡 Analytical Use Cases

1. **Origin-bydel ranking** — which district generates the most departures?
2. **Cross-bydel commuting** — popular routes between districts
3. **Inner city vs outer area** — do behaviours differ?
4. **Geographic visualisation** — bydel aggregates map cleanly to choropleth charts in Power BI

---

## 📋 Prerequisites

1. ✅ Completed Module 3 (Gold Layer)
2. ✅ `silver_trips`, `gold.dim_start_station`, and `gold.dim_end_station` tables exist
