# Module 7 (Advanced): Weather Dimension & Trip-Weather Correlation

## 📖 Overview

This module adds a **new external data source** — daily weather data for Oslo — and integrates it into the existing star schema as a **conformed dimension**. You will then run correlation queries to explore how weather influences bike-sharing behaviour.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand the concept of a **conformed dimension** shared across multiple fact tables
- Fetch real historical weather data from the Open-Meteo free API (no API key required)
- Build `dim_weather` with temperature, precipitation, wind, and derived biking-suitability attributes
- Correlate daily trip volumes with weather conditions

---

## 🎓 Concept: Conformed Dimensions

A **conformed dimension** is a dimension table that:
- Uses the **same grain and same surrogate key** across multiple fact tables
- Can be joined to any fact table in the warehouse without extra transformation
- Is maintained centrally by the data engineering team

`dim_weather` conforms to `dim_date` by sharing `date_key` (format `YYYYMMDD` INT):

```
dim_date ◄──── fact_trips (via date_key)
    ▲
    │  same date_key
dim_weather ◄── join to any fact table via date_key
```

This means adding weather analysis to an **existing** fact table requires only a single additional `JOIN dim_weather dw ON dd.date_key = dw.date_key` — no new keys, no ETL changes to the fact table.

---

## 🌤️ Data Source: Open-Meteo

| Property | Value |
|----------|-------|
| API | [Open-Meteo Historical Archive](https://archive-api.open-meteo.com) |
| Cost | Free, no API key required |
| Coverage | Global, from 1940 onwards |
| Variables used | `temperature_2m_max/min/mean`, `precipitation_sum`, `wind_speed_10m_max`, `weather_code` |
| Location | Oslo (lat=59.9139, lon=10.7522) |

The notebook includes a **synthetic fallback** using Oslo climate normals, so it works even without internet access.

---

## 🗂️ Table Design: `dim_weather`

**Grain:** One row per calendar day.  
**Conforms to:** `dim_date` via `date_key`.

| Column | Type | Description |
|--------|------|-------------|
| `date_key` | INT | Surrogate / conforming key (YYYYMMDD) |
| `full_date` | DATE | Calendar date |
| `temp_max_celsius` | DOUBLE | Daily maximum temperature |
| `temp_min_celsius` | DOUBLE | Daily minimum temperature |
| `temp_mean_celsius` | DOUBLE | Daily mean temperature |
| `temp_range_celsius` | DOUBLE | Max − Min (daily spread) |
| `precipitation_mm` | DOUBLE | Total daily precipitation |
| `wind_speed_kmh` | DOUBLE | Maximum wind speed |
| `weather_code` | INT | WMO weather code |
| `weather_condition` | STRING | Full description (e.g. "Rain") |
| `weather_category` | STRING | Simplified: Sunny / Cloudy / Rainy / Snowy |
| `is_good_biking_weather` | BOOLEAN | `temp > 10°C AND precip < 3mm AND wind < 30 km/h` |
| `temp_bucket` | STRING | Freezing / Cold / Cool / Mild / Warm / Hot |

### What is a WMO Weather Code?

The World Meteorological Organisation (WMO) defines a standard set of numeric codes for weather conditions. Open-Meteo returns these codes, and we decode them to human-readable strings in the dimension:

| Code range | Condition |
|------------|-----------|
| 0 | Clear sky |
| 1–3 | Mainly clear / Partly cloudy |
| 45, 48 | Foggy |
| 51–57 | Drizzle |
| 61–67 | Rain |
| 71–77 | Snow |
| 80–82 | Rain showers |
| 95, 96, 99 | Thunderstorm |

---

## 🔗 Star Schema Diagram

```
              ┌─────────────────┐
              │    dim_weather   │
              │─────────────────│
              │ date_key (PK)   │◄──── conforms to dim_date
              │ full_date       │
              │ temp_mean       │
              │ precipitation   │
              │ wind_speed      │
              │ weather_category│
              │ is_good_biking  │
              │ temp_bucket     │
              └────────┬────────┘
                       │ (join on date_key)
              ┌────────┴────────┐
              │    dim_date     │
              │─────────────────│
              │ date_key (PK)   │
              │ full_date       │
              │ is_weekend      │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              │   fact_trips    │
              │ (or silver_trips│
              │  for this module│
              └─────────────────┘
```

---

## 💡 Analytical Use Cases

1. **Weather-volume correlation** — do more trips happen on sunny days?
2. **Weather sensitivity by day type** — are weekenders more deterred by rain than commuters?
3. **Temperature elasticity** — at what temperature does ridership peak?
4. **Seasonal patterns** — combine with `dim_date.month` to see summer vs winter behaviour
5. **Good biking days ranking** — rank months by proportion of good biking days

---

## 📋 Prerequisites

1. ✅ Completed Module 3 (Gold Layer)
2. ✅ `silver_trips`, `dim_date`, and `dim_station` tables exist
3. ✅ Internet access OR acceptance of synthetic weather fallback
