# Data Modelling & Dimensional Models - Hands-On Course

## 🎯 Course Overview

This hands-on course teaches you how to build a complete data lakehouse solution using Microsoft Fabric. You'll learn data modelling concepts and dimensional model design by transforming real-world bike-sharing data from Oslo City Bike.

### What You'll Learn

- **Medallion Architecture**: Bronze → Silver → Gold data processing pattern
- **Dimensional Modelling**: Fact tables, dimension tables, and star schemas
- **Microsoft Fabric**: Lakehouse, Notebooks, SQL analytics
- **Data Engineering**: Data quality, transformations, and best practices

### Prerequisites

- Access to Microsoft Fabric workspace
- Basic understanding of SQL
- Familiarity with data concepts (tables, columns, data types)

---

## 📊 The Dataset

We're using **Oslo City Bike** open data - real anonymized trip data from Oslo's bike-sharing system.

**Source**: [Oslo Bysykkel Historical Data](https://oslobysykkel.no/en/open-data/historical)

### Data Schema

| Column | Type | Description |
|--------|------|-------------|
| `started_at` | Timestamp | When the trip started |
| `ended_at` | Timestamp | When the trip ended |
| `duration` | Integer | Trip duration in seconds |
| `start_station_id` | String | Unique ID for start station |
| `start_station_name` | String | Name of start station |
| `start_station_description` | String | Location description of start station |
| `start_station_latitude` | Decimal | Latitude of start station (WGS84) |
| `start_station_longitude` | Decimal | Longitude of start station (WGS84) |
| `end_station_id` | String | Unique ID for end station |
| `end_station_name` | String | Name of end station |
| `end_station_description` | String | Location description of end station |
| `end_station_latitude` | Decimal | Latitude of end station (WGS84) |
| `end_station_longitude` | Decimal | Longitude of end station (WGS84) |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MICROSOFT FABRIC                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────┐      ┌───────────┐      ┌───────────┐                │
│  │  BRONZE   │ ───► │  SILVER   │ ───► │   GOLD    │                │
│  │           │      │           │      │           │                │
│  │ Raw CSV   │      │ Cleaned   │      │ Star      │                │
│  │ Files     │      │ Delta     │      │ Schema    │                │
│  │           │      │ Tables    │      │           │                │
│  └───────────┘      └───────────┘      └───────────┘                │
│       │                   │                  │                       │
│       │                   │                  │                       │
│       └───────────────────┴──────────────────┘                       │
│                           │                                          │
│                    ┌──────▼──────┐                                   │
│                    │   SEMANTIC  │                                   │
│                    │    MODEL    │                                   │
│                    └─────────────┘                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Course Structure

| Module | Documentation | Notebook | Description |
|--------|---------------|----------|-------------|
| 0 | [Medallion Architecture](docs/00-medallion-architecture.md) | - | Understand the Bronze/Silver/Gold pattern |
| 1 | [Bronze Layer](docs/01-bronze-layer.md) | `01-bronze-ingestion.ipynb` | Download and ingest raw data |
| 2 | [Silver Layer](docs/02-silver-layer.md) | `02-silver-transformation.ipynb` | Clean and transform data |
| 3 | [Gold Layer](docs/03-gold-layer.md) | `03-gold-dimensional-model.ipynb` | Build dimensional model |

### 🚀 Advanced Modules

After completing Modules 0–3, continue in the [`advanced/`](advanced/README.md) folder:

| Module | Documentation | Notebook | Description |
|--------|---------------|----------|-------------|
| 4 | [Periodic Snapshot](advanced/docs/04-periodic-snapshot.md) | `04-periodic-snapshot.ipynb` | Daily station activity snapshot fact table |
| 5 | [Accumulating Snapshot](advanced/docs/05-accumulating-snapshot.md) | `05-accumulating-snapshot.ipynb` | Trip lifecycle fact table with milestone tracking |
| 6 | [Station Location & Bydel](advanced/docs/06-dim-station-location.md) | `06-dim-station-location.ipynb` | Geographic dimension with Oslo district (bydel) |
| 7 | [Weather Dimension](advanced/docs/07-dim-weather.md) | `07-dim-weather.ipynb` | Weather data source + trip-weather correlation |

---

## 🚀 Getting Started

### Step 1: Set Up Your Lakehouse

1. Open your Microsoft Fabric workspace
2. Create a new **Lakehouse** named `oslo_bysykkel_lakehouse`
3. This lakehouse will store all your Bronze, Silver, and Gold data

### Step 2: Create the Folder Structure

In your Lakehouse, create the following folder structure under `Files`:

```
Files/
├── bronze/
│   └── trips/          # Raw CSV files
├── silver/
│   └── trips/          # Cleaned Delta tables
└── gold/
    ├── fact/           # Fact tables
    └── dim/            # Dimension tables
```

### Step 3: Follow the Modules

Work through each module in order:

1. **Read the documentation** to understand the concepts
2. **Run the notebook** to implement the solution
3. **Verify the results** in your Lakehouse

---

## 🎓 Learning Objectives by Module

### Module 0: Medallion Architecture
- Understand the purpose of each layer
- Learn data quality principles
- Know when to use which layer

### Module 1: Bronze Layer
- Download data from external sources
- Store raw data preserving original format
- Implement data lineage tracking

### Module 2: Silver Layer
- Cleanse and standardize data
- Handle data quality issues
- Create reusable datasets

### Module 3: Gold Layer
- Design star schema models
- Create fact and dimension tables
- Optimize for analytical queries

---

## 📈 Final Dimensional Model

By the end of this course, you'll have built this star schema:

```
                    ┌─────────────────┐
                    │    DimDate      │
                    │─────────────────│
                    │ date_key (PK)   │
                    │ full_date       │
                    │ year            │
                    │ month           │
                    │ day             │
                    │ day_of_week     │
                    │ week_of_year    │
                    │ is_weekend      │
                    └────────┬────────┘
                             │
┌─────────────────┐          │          ┌─────────────────┐
│   DimStation    │          │          │    DimTime      │
│─────────────────│          │          │─────────────────│
│ station_key(PK) │          │          │ time_key (PK)   │
│ station_id      │    ┌─────┴─────┐    │ hour            │
│ station_name    │◄───┤ FactTrips │───►│ minute          │
│ description     │    │───────────│    │ time_of_day     │
│ latitude        │    │ trip_key  │    │ is_rush_hour    │
│ longitude       │    │ date_key  │    └─────────────────┘
└─────────────────┘    │ time_key  │
                       │ start_sk  │
                       │ end_sk    │
                       │ duration  │
                       │ trip_count│
                       └───────────┘
```

---

## 💡 Tips for Success

1. **Take your time**: Understanding concepts is more important than speed
2. **Experiment**: Try modifying the code to see what happens
3. **Ask questions**: Use the documentation and comments in the notebooks
4. **Practice**: The best way to learn is by doing

---

## 📚 Additional Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- [Medallion Architecture - Databricks](https://www.databricks.com/glossary/medallion-architecture)
- [Dimensional Modeling - Kimball Group](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)
- [Oslo City Bike Open Data](https://oslobysykkel.no/en/open-data/historical)

---

**Happy Learning! 🚴‍♀️**
