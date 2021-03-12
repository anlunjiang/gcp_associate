# Cloud Composer: Copying BigQuery Tables Across Different Locations

## Core concepts

* DAG - A Directed Acyclic Graph is a collection of tasks, organised to reflect their relationships and dependencies.
* Operator - The description of a single task, it is usually atomic. For example, the BashOperator is used to execute bash command.
* Task - A parameterised instance of an Operator; a node in the DAG.
* Task Instance - A specific run of a task; characterised as: a DAG, a Task, and a point in time. It has an indicative state: running, success, failed, skipped, ...

`DummyOperator`: Creates Start and End dummy tasks for better visual representation of the DAG.  
`BigQueryToCloudStorageOperator`: Exports BigQuery tables to Cloud Storage buckets using Avro format.  
`GoogleCloudStorageToGoogleCloudStorageOperator`: Copies files across Cloud Storage buckets.  
`GoogleCloudStorageToBigQueryOperator`: Imports tables from Avro files in Cloud Storage bucket.  

* Cloud Composer uses Cloud Storage to store Apache Airflow DAGs, also known as workflows. Each environment has an associated Cloud Storage bucket. Cloud Composer schedules only the DAGs in the Cloud Storage bucket

* Airflow variables are an Airflow-specific concept that is distinct from environment variables

You can set them like: 

    gcloud composer environments run ENVIRONMENT_NAME \
    --location LOCATION variables -- \
    --set KEY VALUE

* When you upload your DAG file to the DAGs folder in Cloud Storage, Cloud Composer parses the file.
    * If no errors are found, the name of the workflow appears in the DAG listing, and the workflow is queued to run immediately if the schedule conditions are met, in this case, None as per the settings.

https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/composer/workflows/bq_copy_across_locations.py

https://cloud.google.com/blog/products/data-analytics/how-to-transfer-bigquery-tables-between-locations-with-cloud-composer

```python
with models.DAG(
        'composer_sample_bq_copy_across_locations',
        default_args=default_args,
        schedule_interval=None) as dag:
    start = dummy_operator.DummyOperator(
        task_id='start',
        trigger_rule='all_success'
    )

    end = dummy_operator.DummyOperator(
        task_id='end',
        trigger_rule='all_success'
    )

    # Get the table list from master file
    all_records = read_table_list(table_list_file_path)

    # Loop over each record in the 'all_records' python list to build up
    # Airflow tasks
    for record in all_records:
        logger.info('Generating tasks to transfer table: {}'.format(record))

        table_source = record['table_source']
        table_dest = record['table_dest']

        BQ_to_GCS = bigquery_to_gcs.BigQueryToCloudStorageOperator(
            # Replace ":" with valid character for Airflow task
            task_id='{}_BQ_to_GCS'.format(table_source.replace(":", "_")),
            source_project_dataset_table=table_source,
            destination_cloud_storage_uris=['{}-*.avro'.format(
                'gs://' + source_bucket + '/' + table_source)],
            export_format='AVRO'
        )

        GCS_to_GCS = gcs_to_gcs.GoogleCloudStorageToGoogleCloudStorageOperator(
            # Replace ":" with valid character for Airflow task
            task_id='{}_GCS_to_GCS'.format(table_source.replace(":", "_")),
            source_bucket=source_bucket,
            source_object='{}-*.avro'.format(table_source),
            destination_bucket=dest_bucket,
            # destination_object='{}-*.avro'.format(table_dest)
        )

        GCS_to_BQ = gcs_to_bq.GoogleCloudStorageToBigQueryOperator(
            # Replace ":" with valid character for Airflow task
            task_id='{}_GCS_to_BQ'.format(table_dest.replace(":", "_")),
            bucket=dest_bucket,
            source_objects=['{}-*.avro'.format(table_source)],
            destination_project_dataset_table=table_dest,
            source_format='AVRO',
            write_disposition='WRITE_TRUNCATE',
            autodetect=True
        )

        start >> BQ_to_GCS >> GCS_to_GCS >> GCS_to_BQ >> end
```