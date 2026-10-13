# Building an End-to-End Data Engineering Solution with Azure ✨

This project demonstrates an end-to-end data engineering pipeline using Azure services. It ingests the AdventureWorks dataset from GitHub, stores raw data in Azure Data Lake Storage Gen2 (ADLS Gen2), transforms it with Azure Databricks, exposes curated data through Azure Synapse Analytics, and supports reporting with Power BI.

## Project Architecture

![Project architecture](images/image%201.png)

## Architecture Overview

### Azure services used

- **Azure Data Factory (ADF):** Ingests data and orchestrates pipeline activities.
- **Azure Data Lake Storage Gen2:** Stores data in Bronze, Silver, and Gold layers.
- **Azure Databricks:** Reads raw data, cleans and transforms it, and writes structured output.
- **Azure Synapse Analytics:** Queries curated data using SQL and provides a serving layer for analytics.
- **Power BI:** Connects to reporting-ready data to create dashboards and business insights.

> **Note:** Bronze, Silver, and Gold are logical data layers. They may be separate containers or folders, depending on your storage configuration.

## Step 1: Setting Up the Azure Environment ⚙️

The Azure resources used in this project are:

1. **Azure Data Factory** — data ingestion, orchestration, and automation.
2. **Azure Storage Account with ADLS Gen2** — storage for Bronze (raw), Silver (cleaned), and Gold (curated) data.
3. **Azure Databricks workspace** — data transformation and processing.
4. **Azure Synapse Analytics workspace** — SQL-based querying and analytics.

Configure authentication and access permissions for each service. Use managed identities where supported and grant only the required permissions.

![Azure resources](images/image%202.jpg)

## Step 2: Implementing the Data Pipeline Using ADF 🚀

Azure Data Factory orchestrates the pipeline and copies source files from GitHub into the Bronze layer of ADLS Gen2.

### Pipeline activities

1. **Lookup activity**
   - Reads a configuration list containing source file names or URLs.
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

![Azure Data Factory pipeline](images/image%203.jpg)

## Step 3: Storing Raw Data in the Bronze Layer

The Bronze layer stores the raw files copied from GitHub. Keeping the original data makes it easier to trace issues and rerun transformations.

![Bronze container with raw data](images/image%204.jpg)

## Step 4: Data Transformation with Azure Databricks 🔄

Azure Databricks reads raw files from Bronze, transforms them into a consistent structure, and writes the processed data to Silver. Additional business-ready aggregations can be written to Gold.

### Key steps

1. **Compute setup**
   - Create or select Databricks compute suitable for the workload.
   - Start the compute before running notebooks or jobs.

2. **Data Lake access**
   - Configure secure access to ADLS Gen2.
   - Use the correct storage path and permissions.

3. **Read Bronze data**
   - Load source files into Spark DataFrames.
   - Inspect schemas, record counts, and sample rows.

4. **Clean and transform**
   - Handle nulls and duplicates where appropriate.
   - Apply consistent data types and column names.
   - Validate records and apply required business rules.

5. **Write transformed data**
   - Save cleaned data to Silver.
   - Write curated outputs or aggregations to Gold when required.

![Azure Databricks compute and transformation](images/image%205.jpg)

### Medallion architecture

| Layer | Purpose |
|---|---|
| **Bronze** | Raw source data, preserved for traceability and reprocessing |
| **Silver** | Cleaned, validated, and standardized data |
| **Gold** | Curated, business-ready data for analytics and reporting |

## Step 5: Data Warehousing with Azure Synapse Analytics 📊

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

![Synapse SQL scripts](images/image%206.jpg)

## Step 6: Gold Layer — Reporting-Ready Data

The Gold layer contains curated data intended for business reporting and analytics. Confirm that the expected output files have been written to the correct container or folder before querying them from Synapse.

![Gold container](images/image%207.jpg)

## Step 7: Business Intelligence Integration with Power BI 📈

Power BI connects to the reporting-ready data exposed through Synapse and turns it into dashboards and business insights.

### Reporting steps

1. Connect Power BI to the appropriate Synapse SQL endpoint.
2. Select the reporting views or tables.
3. Define relationships and measures where needed.
4. Build visuals for relevant business metrics.
5. Validate totals against the source data and refresh the report.

![Power BI integration](images/image%208.jpg)

## Step 8: Final Output

The complete pipeline connects ingestion, storage, transformation, SQL analytics, and reporting in one end-to-end workflow.

![Final project output](images/image%209.jpg)

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

## Repository Structure

Make sure the repository has this structure. The image filenames and extensions must match exactly.

```text
Adventure-Works-Azure-Data-Engineering-Project/
├── README.md
└── images/
    ├── image 1.jpg
    ├── image 2.jpg
    ├── image 3.jpg
    ├── image 4.jpg
    ├── image 5.jpg
    ├── image 6.jpg
    ├── image 7.jpg
    ├── image 8.jpg
    └── image 9.jpg
```

> If your uploaded files are named `image1.jpg` (without a space), change the links to `images/image1.jpg`. If they are `.png` files, use `.png` instead of `.jpg`.
