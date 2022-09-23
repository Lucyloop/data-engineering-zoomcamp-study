
# steps to use airflow

1. docker-compose build 
2. docker-compose airflow init
3. docker-compose up
4. docker-compose ps : to check servers 

# airflow workflow
## DAG - gcp
download_dataset -> format_to_parquet -> local_to_gcs -> bigquery_external_table

# ingestion with airflow
