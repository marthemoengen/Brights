# Module 7 (Advanced): Weather Dimension & Trip-Weather Correlation

## 📖 Overview

This module adds a **new external data source** — daily weather data for Oslo — and integrates it into the existing star schema as a daily weather dimension. You will then run correlation queries to explore how weather influences bike-sharing behaviour.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand how a daily dimension can connect directly to a fact table through a shared key
- Fetch real historical weather data from the Open-Meteo free API (no API key required)
- Build `gold.dim_weather` with temperature, precipitation, wind, and derived biking-suitability attributes
- Correlate daily trip volumes with weather conditions

---

## 🎓 Concept: Weather as a Fact-Linked Dimension

`gold.dim_weather` has one row per calendar day. It uses `date_key`, the same daily key already stored in `gold.fact_trips`.

The important relationship is between weather and the fact table:

```
gold.dim_weather ── date_key ──► gold.fact_trips
                                      │
                                      ├──► gold.dim_date
                                      ├──► gold.dim_time
                                      ├──► gold.dim_start_station
                                      └──► gold.dim_end_station
```

`gold.dim_date` is still useful for calendar attributes such as month, weekday, and weekend flags. But weather does not need to connect through `gold.dim_date`; it can join directly to `gold.fact_trips` using `date_key`.

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

## 🗂️ Table Design: `gold.dim_weather`

**Grain:** One row per calendar day.  
**Connects to:** `gold.fact_trips` via `date_key`.

| Column | Type | Description |
|--------|------|-------------|
| `date_key` | INT | Daily key used to join directly to fact tables (YYYYMMDD) |
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
              │ gold.dim_weather │
              │─────────────────│
              │ date_key (PK)   │
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
              │ gold.fact_trips │
              │────────────────│
              │ date_key       │
              │ duration       │
              │ trip_count     │
              └────────┬───────┘
                       │
              ┌────────┴────────┐
              │ gold.dim_date   │
              │ calendar attrs  │
              └─────────────────┘
```

---

## 💡 Analytical Use Cases

1. **Weather-volume correlation** — do more trips happen on sunny days?
2. **Weather sensitivity by day type** — are weekenders more deterred by rain than commuters?
3. **Temperature elasticity** — at what temperature does ridership peak?
4. **Seasonal patterns** — join the fact to `gold.dim_date` when you need month or weekend attributes
5. **Good biking days ranking** — rank months by proportion of good biking days

---

## 📋 Prerequisites

1. ✅ Completed Module 3 (Gold Layer)
2. ✅ `gold.fact_trips` and `gold.dim_date` tables exist
3. ✅ Internet access OR acceptance of synthetic weather fallback
