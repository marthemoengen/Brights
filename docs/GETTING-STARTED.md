# Getting Started Guide

## Quick Setup in Microsoft Fabric

Follow these steps to set up your environment and run the course materials.

---

## 🚀 Step 1: Create a Microsoft Account

Microsoft Fabric requires a work or school account. If you do not already have one:

1. Create a free [Microsoft account](https://signup.live.com/) or use an existing organizational account.
2. Go to [Microsoft Fabric](https://app.fabric.microsoft.com/) and sign in.
3. Complete any first-time setup prompts, including selecting your region.

> A personal account can be used to start the trial, but some organizations restrict trials or Fabric features through tenant policy.

---

## ⚡ Step 2: Start the Microsoft Fabric Trial

1. In the Fabric portal, select your profile icon in the top-right corner.
2. Select **Start trial** or **Try Fabric for free**.
3. Review the trial details and select **Start trial** to create your trial capacity.
4. Wait for the confirmation message, then select **Switch** if Fabric asks you to change to the trial capacity.

The Fabric trial is time-limited. This course uses Lakehouse and Notebook items, which require an active Fabric capacity or trial.

### Can't find the trial option?

- Confirm you are signed in with a work or school account.
- Ask your Microsoft 365 or Fabric administrator to enable Fabric trials for your tenant.
- If your organization does not allow trials, ask to be assigned to an existing Fabric capacity with **Contributor** access to a workspace.

---

## 🧭 Step 3: Create or Access a Workspace

1. In the Fabric portal, select **Workspaces** in the navigation pane.
2. Select **+ New workspace**.
3. Name the workspace, for example `oslo-bysykkel-course`.
4. In the workspace settings, assign the workspace to your Fabric trial capacity.
5. Verify that you have **Contributor** or **Admin** permissions.

---

## 🏠 Step 4: Create the Lakehouse

1. In your workspace, click **+ New** → **Lakehouse**
2. Name it: `oslo_bysykkel_lakehouse`
3. Click **Create**

### Verify Lakehouse Structure

Your Lakehouse should have these sections:
- **Tables/** - For Delta Lake tables (Silver & Gold)
- **Files/** - For raw files (Bronze)

---

## 📁 Step 5: Create Folder Structure

In the Lakehouse, create the folder structure for Bronze data:

1. Click on **Files** in the left panel
2. Click **...** → **New subfolder**
3. Create: `bronze`
4. Navigate into `bronze`, create: `trips`

Final structure:
```
Files/
└── bronze/
    └── trips/
```

---

## 📓 Step 6: Create and Run Notebooks

### Create Each Notebook

For each notebook file in this repository:

1. In your workspace, click **+ New** → **Notebook**
2. Name it according to the file name (e.g., `01-bronze-ingestion`)
3. **Attach to Lakehouse**: Click **Add Lakehouse** on the left panel and select `oslo_bysykkel_lakehouse`

### Copy the Code

1. Open the `.ipynb` file from this repository
2. Copy each cell's content into your Fabric notebook
3. For SQL cells, change the cell language to **Spark SQL** using the cell language selector

### Run Order

Execute the notebooks in this order:

| Order | Notebook | Creates |
|-------|----------|---------|
| 1 | `01-bronze-ingestion.ipynb` | Bronze CSV files |
| 2 | `02-silver-transformation.ipynb` | `silver_trips` table |
| 3 | `03-gold-dimensional-model.ipynb` | `dim_date`, `dim_time`, `dim_station`, `fact_trips` |

---

## ✅ Step 7: Verify Your Results

After running all notebooks, verify:

### In Files/
```
Files/
└── bronze/
    └── trips/
        ├── 2026_01.csv
        ├── 2026_02.csv
        ├── 2026_03.csv
        ├── 2026_04.csv
        ├── 2026_05.csv
        ├── 2026_06.csv
        ├── 2026_07.csv
        ├── 2026_08.csv
        └── _ingestion_log.csv
```

### In Tables/
```
Tables/
├── silver_trips
├── dim_date
├── dim_time
├── dim_station
└── fact_trips
```

---

## 🔧 Troubleshooting

### "Lakehouse not attached"
- Click **Add Lakehouse** in the notebook sidebar
- Select your lakehouse

### "File not found" errors
- Verify the bronze/trips folder exists
- Check the previous notebook completed successfully

### "Table already exists"
- Use `CREATE OR REPLACE TABLE` (already in the code)
- Or drop the table first: `DROP TABLE IF EXISTS table_name`

### Notebook cells timeout
- Large data operations may take time
- Check Spark job progress in the notebook UI

---

## 📊 Next Steps After Completion

1. **Explore the Data**: Run ad-hoc SQL queries against your tables
2. **Build Reports**: Create a Power BI report connected to the Gold tables
3. **Create Semantic Model**: Build a semantic model from the star schema
4. **Add More Analysis**: Extend the model with additional measures

---

## 📚 Course Files Reference

| File | Description |
|------|-------------|
| `README.md` | Course overview and introduction |
| `docs/00-medallion-architecture.md` | Medallion pattern explanation |
| `docs/01-bronze-layer.md` | Bronze layer guide |
| `docs/02-silver-layer.md` | Silver layer guide |
| `docs/03-gold-layer.md` | Gold layer and dimensional modelling guide |
| `notebooks/01-bronze-ingestion.ipynb` | Bronze layer notebook |
| `notebooks/02-silver-transformation.ipynb` | Silver layer notebook |
| `notebooks/03-gold-dimensional-model.ipynb` | Gold layer notebook |
