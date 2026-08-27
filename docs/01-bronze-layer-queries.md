# Module 1: Bronze Layer - SQL Endpoint Queries (Optional Variant)

This is the retained answer version for an **optional table-backed Bronze variant**. It is not used by the files-only course path in [01-bronze-layer.md](01-bronze-layer.md).

The standard Bronze notebook creates CSV files only. Those files are not exposed as `dbo` tables by the Lakehouse SQL analytics endpoint. These queries work only if an instructor separately creates a raw Delta table named `dbo.bronze_trips` from the CSV files before running them.

For the standard course path, use the notebook output and ingestion log in Bronze, then perform SQL endpoint exercises after the Silver notebook creates `silver_trips`.

## Prerequisites

- The Bronze ingestion notebook has completed.
- The Lakehouse SQL analytics endpoint is open.
- The raw table `dbo.bronze_trips` exists.

The Bronze table intentionally keeps source values as strings. Do not modify the Bronze data in these exercises. Type conversion and data-quality corrections belong in the Silver layer.

## Open the SQL Endpoint

1. Open `oslo_bysykkel_lakehouse` in the Fabric workspace.
2. Select **SQL analytics endpoint**.
3. Select **New SQL query**.
4. Run each query separately.

## Query 1: Preview the Raw Bronze Table

```sql
SELECT TOP 10 *
FROM dbo.bronze_trips;
```

The records should resemble the source CSV. No cleaned or derived business columns should be present.

## Query 2: Inspect the Schema

```sql
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'bronze_trips'
ORDER BY ORDINAL_POSITION;
```

The source columns should be strings at this stage. Bronze preserves the source representation; Silver is where the pipeline will cast timestamps, durations, and coordinates.

## Query 3: Confirm Ingestion Volume and Station Coverage

```sql
SELECT
    COUNT(*) AS total_trips,
    COUNT(DISTINCT start_station_id) AS unique_start_stations,
    COUNT(DISTINCT end_station_id) AS unique_end_stations
FROM dbo.bronze_trips;
```

Compare `total_trips` with the total printed by the notebook's ingestion verification. The counts should match.

## Query 4: Profile Raw Data Quality

```sql
SELECT 'null_started_at' AS issue, COUNT(*) AS issue_count
FROM dbo.bronze_trips
WHERE started_at IS NULL OR started_at = ''

UNION ALL

SELECT 'null_ended_at', COUNT(*)
FROM dbo.bronze_trips
WHERE ended_at IS NULL OR ended_at = ''

UNION ALL

SELECT 'null_duration', COUNT(*)
FROM dbo.bronze_trips
WHERE duration IS NULL OR duration = ''

UNION ALL

SELECT 'zero_or_negative_duration', COUNT(*)
FROM dbo.bronze_trips
WHERE TRY_CAST(duration AS INT) <= 0;
```

Do not fix these records in Bronze. Record the findings and use them to explain the validation rules implemented in Module 2.

## Expected Learning Outcome

Learners should be able to explain:

- What one Bronze row represents.
- Why the Bronze table preserves raw values.
- How to inspect a Lakehouse table through the SQL endpoint.
- How to measure ingestion completeness.
- Which quality checks belong in the Silver transformation.
