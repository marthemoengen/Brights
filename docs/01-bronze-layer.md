# Module 1: Bronze Layer - Data Ingestion

## 📖 Overview

In this module, you'll download raw trip data from Oslo City Bike and store it in the Bronze layer of your Lakehouse. The goal is to preserve the original data exactly as received.

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand the purpose of the Bronze layer
- Download data from external HTTP sources
- Store raw files in your Lakehouse
- Add metadata for data lineage tracking

---

## 📋 Prerequisites

Before starting this module:

1. ✅ You have a Microsoft Fabric workspace
2. ✅ You have created a Lakehouse named `oslo_bysykkel_lakehouse`
3. ✅ You understand the [Medallion Architecture](00-medallion-architecture.md)

---

## 🔧 Step-by-Step Instructions

### Step 1: Create the Bronze Folder Structure

In your Lakehouse, navigate to **Files** and create the following folder structure:

```
Files/
└── bronze/
    └── trips/
```

**How to create folders:**
1. Open your Lakehouse in Fabric
2. Click on **Files** in the left panel
3. Click the **...** menu and select **New subfolder**
4. Create `bronze`, then navigate into it
5. Create `trips` inside `bronze`

---

### Step 2: Create a New Notebook

1. In your workspace, click **+ New** → **Notebook**
2. Name it `01-bronze-ingestion`
3. Attach it to your Lakehouse:
   - Click **Add Lakehouse** on the left panel
   - Select `oslo_bysykkel_lakehouse`

---

### Step 3: Run the Notebook

Copy the code from `notebooks/01-bronze-ingestion.ipynb` into your Fabric notebook and run each cell.

The notebook will:
1. Define the list of monthly CSV files for 2026
2. Download each file from the Oslo Bysykkel API
3. Save the raw CSV files to `Files/bronze/trips/`
4. Create an ingestion log with metadata
 
The Bronze layer contains files only. CSV files in the Lakehouse **Files** area do not automatically appear as `dbo` tables in the SQL analytics endpoint. The SQL endpoint becomes the testing surface after the Silver notebook reads these files and creates the `silver_trips` Delta table.

For Bronze, use the notebook output, the Lakehouse Files view, and the ingestion log to verify that the expected files, row counts, and file sizes are present. The Silver module contains the SQL endpoint exercises for the typed and validated data.

---

## 📝 Understanding the Code

### Downloading Files

The notebook uses Python's `requests` library to download files:

```python
import requests

url = "https://data.urbansharing.com/oslobysykkel.no/trips/v1/2026/01.csv"
response = requests.get(url)
```

### Saving to Lakehouse

Files are saved using the Fabric file system path:

```python
file_path = "/lakehouse/default/Files/bronze/trips/2026_01.csv"
with open(file_path, 'wb') as f:
    f.write(response.content)
```

### Adding Metadata

For data lineage, we track:
- **Source URL**: Where the data came from
- **Ingestion Timestamp**: When it was downloaded
- **File Size**: Original file size in bytes
- **Row Count**: Number of records

---

## ✅ Verification Checklist

After completing this module, verify:

| Check | How to Verify |
|-------|---------------|
| ✅ Bronze folder exists | Navigate to Files/bronze/trips/ |
| ✅ CSV files downloaded | See 8 CSV files (01.csv to 08.csv) |
| ✅ Files have content | Click a file to preview data |
| ✅ Ingestion log created | Check Files/bronze/trips/_ingestion_log.csv |

---

## 📊 Expected Results

### Files in Bronze Layer

```
Files/bronze/trips/
├── 2026_01.csv    # January 2026 trips
├── 2026_02.csv    # February 2026 trips
├── 2026_03.csv    # March 2026 trips
├── 2026_04.csv    # April 2026 trips
├── 2026_05.csv    # May 2026 trips
├── 2026_06.csv    # June 2026 trips
├── 2026_07.csv    # July 2026 trips
├── 2026_08.csv    # August 2026 trips (partial, updated daily)
└── _ingestion_log.csv
```

### Sample Ingestion Log

| source_file | source_url | ingestion_time | file_size_bytes | row_count |
|-------------|------------|----------------|-----------------|-----------|
| 2026_01.csv | https://... | 2026-08-25 10:00:00 | 15234567 | 125432 |
| 2026_02.csv | https://... | 2026-08-25 10:00:05 | 18765432 | 156789 |

---

## 🔍 Raw Data Preview

Use the Lakehouse Files view to preview the raw data. The columns look like this:

| Column | Example Value |
|--------|---------------|
| started_at | 2026-01-15 08:32:45.123 |
| ended_at | 2026-01-15 08:47:12.456 |
| duration | 867 |
| start_station_id | 623 |
| start_station_name | Majorstuen T-bane |
| start_station_description | Ved T-banestasjonen |
| start_station_latitude | 59.9298 |
| start_station_longitude | 10.7135 |
| end_station_id | 412 |
| end_station_name | Nationaltheatret |
| end_station_description | Stortingsgata |
| end_station_latitude | 59.9134 |
| end_station_longitude | 10.7328 |

---

## ⚠️ Troubleshooting

### Common Issues

| Problem | Solution |
|---------|----------|
| "Folder not found" error | Create the bronze/trips folder manually first |
| Download timeout | Run the cell again; network issues are temporary |
| "Permission denied" | Ensure notebook is attached to the Lakehouse |
| Empty files | Check if the URL is accessible in a browser |

---

## 🎓 Key Takeaways

1. **Bronze layer preserves raw data** - No transformations applied
2. **Metadata is crucial** - Track when and where data came from
3. **File naming matters** - Use consistent, descriptive names
4. **Error handling** - Always plan for network/download failures
5. **Bronze is file-based** - Use the ingestion log and notebook output to verify the files before transforming them

---

## 🏃‍♂️ Next Steps

Your raw data is now safely stored in the Bronze layer. Next, you'll clean and transform this data in the Silver layer.

👉 **[Module 2: Silver Layer](02-silver-layer.md)** - Clean and transform data

---

## 📚 Additional Resources

- [Microsoft Fabric: Working with Files](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-notebook-load-data)
- [Oslo City Bike Data Documentation](https://oslobysykkel.no/en/open-data/historical)
- [Python Requests Library](https://docs.python-requests.org/)
