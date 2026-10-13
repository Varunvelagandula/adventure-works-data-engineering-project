# Building an End-to-End Data Engineering Solution with Azure ✨

This project demonstrates an end-to-end data engineering pipeline using Azure services. It ingests the AdventureWorks dataset from GitHub, stores raw data in Azure Data Lake Storage Gen2 (ADLS Gen2), transforms it with Azure Databricks, exposes curated data through Azure Synapse Analytics, and supports reporting with Power BI.

![Project architecture](images/image 1.jpg)
## Architecture Overview

### Azure services used

- **Azure Data Factory (ADF):** Ingests data and orchestrates pipeline activities.
- **Azure Data Lake Storage Gen2:** Stores data in Bronze, Silver, and Gold layers.
- **Azure Databricks:** Reads raw data, cleans and transforms it, and writes structured output.
- **Azure Synapse Analytics:** Queries curated data using SQL and provides a serving layer for analytics.
- **Power BI:** Connects to reporting-ready data to create dashboards and business insights.

> **Note:** Bronze, Silver, and Gold are logical data layers. They can be separate containers or folders, depending on how the storage account is configured.

## Step 1: Setting Up the Azure Environment ⚙️

The following Azure resources are used in the solution:

1. **Azure Data Factory** — data ingestion, orchestration, and automation.
2. **Azure Storage Account with ADLS Gen2** — storage for Bronze (raw), Silver (cleaned), and Gold (curated) data.
3. **Azure Databricks workspace** — data transformation and processing.
4. **Azure Synapse Analytics workspace** — SQL-based querying and analytics.

Configure authentication and access permissions for each service. For production workloads, prefer managed identities and grant only the required permissions.

**Screenshot — Azure resources:**  
Add your Azure resource-group screenshot here.

```markdown
![Azure resources in the resource group](images/azure-resources.png)
```

## Step 2: Implementing the Data Pipeline Using ADF 🚀

Azure Data Factory orchestrates the pipeline and copies the source files from GitHub into the Bronze layer of ADLS Gen2.

### Pipeline activities

1. **Lookup activity**
   - Reads a configuration list containing the source file names or URLs.
   - Supplies the list of files that need to be processed.

2. **ForEach activity**
   - Iterates through the items returned by Lookup.
   - Runs the copy operation for each item.

3. **Dynamic Copy activity**
   - Uses parameterized source and destination settings.
   - Reads each configured file from GitHub through an HTTP source.
   - Writes the downloaded data into the Bronze container.

4. **Pipeline parameters**
   - Make the pipeline reusable when file names or source paths change.
   - Validate the URL, relative path, file name, and destination path for every item.

**Expected flow:**

```text
Lookup (file configuration)
        ↓
ForEach (each file)
        ↓
Copy activity (GitHub / HTTP → ADLS Gen2 Bronze)
```

After a successful run, verify that the expected files exist in the Bronze container and that the copied files have non-zero sizes.

**Screenshot — ADF pipeline:**  
Add your pipeline screenshot here.

```markdown
![ADF Lookup, ForEach and Copy activities](images/adf-pipeline.png)
```

**Screenshot — Bronze container:**  
Add your Bronze container screenshot here.

```markdown
![Raw AdventureWorks files in Bronze](images/bronze-container.png)
```

## Step 3: Data Transformation with Azure Databricks 🔄

Azure Databricks reads the raw files from Bronze, transforms them into a consistent structure, and writes the processed data to Silver. Additional business-ready aggregations can then be written to Gold.

### Key steps

1. **Compute setup**
   - Create or select Databricks compute suitable for the workload.
   - Start the compute before running notebooks or jobs.

2. **Data Lake access**
   - Configure secure access to ADLS Gen2.
   - Use the appropriate storage path and permissions.

3. **Read Bronze data**
   - Load the source files into Spark DataFrames.
   - Inspect schemas, record counts, and sample rows.

4. **Clean and transform**
   - Handle nulls and duplicates where appropriate.
   - Apply consistent data types and column names.
   - Validate records and apply required business rules.

5. **Write transformed data**
   - Save cleaned data to the Silver layer.
   - Write curated outputs or aggregations to Gold when required.

### Medallion layers

| Layer | Purpose |
|---|---|
| **Bronze** | Raw source data, preserved for traceability and reprocessing |
| **Silver** | Cleaned, validated, and standardized data |
| **Gold** | Curated, business-ready data for analytics and reporting |

**Screenshot — Databricks compute or notebook:**  
Add your Databricks screenshot here.

```markdown
![Databricks compute and transformation](images/databricks.png)
```

## Step 4: Data Warehousing with Azure Synapse Analytics 📊

Azure Synapse Analytics provides the SQL layer used to query curated data for analytics. In this implementation, serverless SQL can query files in ADLS Gen2 without requiring a dedicated SQL pool.

### Main steps

1. **Configure access to ADLS Gen2**
   - Give the Synapse workspace identity the required storage permissions.
   - Confirm the storage URL, container, and folder paths.

2. **Create a database and schema**
   - Organize external data objects in a database and schema.

3. **Create an external data source**
   - Point Synapse SQL to the appropriate ADLS Gen2 location.
   - Configure credentials or identity-based access correctly.

4. **Create an external file format**
   - Define the format used by the files, such as Parquet or CSV.

5. **Create external tables and views**
   - Define external tables over the curated files.
   - Create views to simplify reporting queries and expose useful business fields.

6. **Validate the results**
   - Run test queries and confirm that the expected rows are returned.
   - Check that the external table location matches the actual folder and file format.

### Example SQL pattern

Adapt the database, schema, storage URL, credentials, folder paths, and file format to match your own environment. Do not paste secrets into a public repository.

```sql
-- Example only: use your actual database and schema.
CREATE SCHEMA gold;
GO

-- Create an external data source after configuring the
-- required credential/identity and storage permissions.
CREATE EXTERNAL DATA SOURCE source_gold
WITH (
    LOCATION = 'https://<storage-account>.dfs.core.windows.net/<container>'
);
GO

CREATE EXTERNAL FILE FORMAT format_parquet
WITH (
    FORMAT_TYPE = PARQUET
);
GO

-- Example only: the folder must exist and contain compatible files.
CREATE EXTERNAL TABLE gold.example_table
(
    example_id INT
)
WITH
(
    LOCATION = 'example-folder/',
    DATA_SOURCE = source_gold,
    FILE_FORMAT = format_parquet
);
GO
```

> **Important:** The example SQL is a template, not a complete drop-in script. External-table column definitions must match the actual files. If your Gold files are Parquet, use a compatible Parquet external file format and the correct folder path.

**Screenshot — Synapse SQL scripts:**  
Add your Synapse SQL editor screenshot here.

```markdown
![Synapse external data source and tables](images/synapse-sql.png)
```

**Screenshot — Gold container:**  
Add your Gold container screenshot here.

```markdown
![Curated data in the Gold layer](images/gold-container.png)
```

## Step 5: Business Intelligence Integration 📈

Power BI connects to the reporting-ready data exposed through Synapse and turns it into dashboards and business insights.

### Reporting steps

1. Connect Power BI to the appropriate Synapse SQL endpoint.
2. Select the reporting views or tables.
3. Define relationships and measures where needed.
4. Build visuals for relevant business metrics.
5. Validate totals against the source data and refresh the report.

**Screenshot — Power BI dashboard:**  
Add your Power BI screenshot here.

```markdown
![AdventureWorks Power BI dashboard](images/power-bi-dashboard.png)
```

## End-to-End Data Flow

```text
AdventureWorks files on GitHub
             ↓
    Azure Data Factory
             ↓
      ADLS Gen2 Bronze
             ↓
      Azure Databricks
             ↓
      ADLS Gen2 Silver
             ↓
       ADLS Gen2 Gold
             ↓
  Azure Synapse Analytics
             ↓
           Power BI
```

## Security and Reliability

- Use managed identities where supported and grant least-privilege access.
- Keep credentials and secrets out of notebooks, pipeline parameters, and GitHub.
- Validate source URLs and destination paths before running the pipeline.
- Monitor ADF pipeline runs and investigate failed Copy activities individually.
- Check file counts, sizes, schemas, and row counts after ingestion and transformation.
- Make pipeline runs repeatable and avoid duplicate outputs when rerunning jobs.
- Configure Power BI refresh and permissions according to the intended audience.

## Conclusion

This project demonstrates how Azure services can work together to build a practical data engineering solution: ADF handles ingestion and orchestration, ADLS Gen2 stores the medallion layers, Databricks transforms the data, Synapse exposes it for SQL analytics, and Power BI presents business insights.

## Repository Screenshots

To display your own screenshots in this README:

1. Create a folder named `images` in the root of your GitHub repository.
2. Upload your screenshot image files to that folder.
3. Rename the files to match the paths used above, or update the Markdown image paths.
4. Commit and push the updated `README.md` and `images` folder.

Example repository layout:

```text
Adventure-Works-Azure-Data-Engineering-Project/
├── README.md
└── images/
    ├── azure-resources.png
    ├── adf-pipeline.png
    ├── bronze-container.png
    ├── databricks.png
    ├── synapse-sql.png
    ├── gold-container.png
    └── power-bi-dashboard.png
```
