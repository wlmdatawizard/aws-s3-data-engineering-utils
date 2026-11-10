# MonsterForge | AWS Data Engineering Pipeline

MonsterForge is a batch data engineering project that turns messy monster inventory CSVs into clean, queryable records. A Streamlit app guides the workflow from file upload through local PySpark processing, Amazon S3 storage, AWS Glue cataloging, and Amazon Athena queries.

**Stack:** Python · PySpark · pandas · Streamlit · boto3 · Amazon S3 · AWS Glue · Amazon Athena

## How it works

1. **Upload raw data.** The app saves the source CSV to S3 under a run ID and updates the `latest` copy.
2. **Transform and validate.** Local PySpark checks for required columns, normalizes categories, parses dates and prices, and adds source file and ingestion metadata.
3. **Quarantine invalid records.** Rows with a missing monster ID or name, an invalid date, or an invalid danger level receive a quarantine reason. The quality report tracks duplicate IDs separately.
4. **Publish the outputs.** The app writes clean and quarantined CSVs and a JSON quality report to S3. Each has a run-specific copy and a `latest` copy. The app also writes separate CSVs for querying.
5. **Catalog and query.** The app provides controls for starting and checking Glue crawlers. Its Athena page can list tables, preview rows, count records, and run SQL.

## Explore the code

| File | Purpose |
| --- | --- |
| [`app_corrected.py`](app_corrected.py) | Streamlit workflow for uploads, ETL, crawler controls, and Athena queries |
| [`src/monsterforge_etl.py`](src/monsterforge_etl.py) | Input checks, transformations, quarantine rules, and quality reporting |
| [`src/monsterforge_glue_setup.py`](src/monsterforge_glue_setup.py) | Glue database and crawler setup |
| [`src/athena_operations.py`](src/athena_operations.py) | Athena query helpers |

## Running the project

The app requires Python, Java for local Spark, AWS credentials, and an S3 bucket. The catalog and query steps also require Glue and Athena permissions. The current dependency file covers the original S3 utilities. Install `streamlit` and `pyspark` alongside those dependencies to use the app.

The app and Glue setup script currently reference project-specific AWS settings. Before running them in your own account, update the bucket, region, Glue role, and catalog settings as applicable. The Glue crawler targets should match the S3 prefixes you want Athena to query.

From the repository root in PowerShell:

```powershell
python -m pip install -r requirements.txt
python -m pip install streamlit pyspark
$env:PYTHONPATH = "$PWD\src"
python -m streamlit run app_corrected.py
```

The app imports a helper from `src` before adding that directory to its import path, so the launch commands set `PYTHONPATH` explicitly.