# Weakpoints and Technology Summaries

## Processing
* `Dataproc`
    * Runs on Spark cluster to process
    * Create separate Dataproc clusters for different purposes - since compute is separate to storage 
    * Google recommends Cloud Storage instead of HDFS as it is much more cost effective especially when jobs aren’t running.
    * Separation of compute and storage
* `Dataflow` 
    * Uses Apache beam to process 
    * You can use Dataflow to write to bq and bt. Bigtable and BigQueryIO transforms can be used in the Pipeline
    * Use Combine instead of GroupBy to avoid shuffling
    * Dataflow templates enable non devs to create pipelines
 
## Pre-Post Processing
* `DataFusion`
    * Create a Spark pipeline and run it on Dataproc cluster
    * More designed for data ingestion from one source to another one
* `DataPrep`
    * Create a Beam pipeline and run it on Dataflow
    * More designed for data preparation (as its name means), data cleaning, new column creation, splitting column
    * Has recipes that records your data edits

## Storage
* `BigTable`
    * Cloud BigTable CBT - The cbt tool is a command-line interface for performing several different operations on Cloud Bigtable, preferred over HBase
* `BigQuery`
    * Instead of using `COUNT(DISTINCT) ` use `APPROX_COUNT_DISTINCT` on large data streams when a small uncertainty is tolerable.
    * Bigquery supports wildcard syntax when querying from multiple table objects e.g. "`bigquery-public-data.noaa_gsod.gsod*`"
    * wildcard bigquery standard sql for multiple tables is `_TABLE_SUFFIX`
    * Bigquery is acid compliant
    * bq supports both batch and stream - but batch is free - streaming is charged by size
    * Changing datasets to different location: Datasets are immutable - so location cannot be changed. 
        * To duplicate
            * you need to export data into a nearby bucket on GCS
            * Copy to a bucket in the region you want your new dataset to be.
            * Then import the new dataset in the new location
        * The reason to swap buckets is that BigQuery can only read and write from nearby buckets of the datasets
        * There is no cross-location move/copy command on bigquery
    * Federated or external bigquery tables are useful for ETL pipelines according to google best practices
* `DataStore`
    * NoSQL database
    * Data are properties contained in an entity in kind categories
* `Storage Transfer Services`
    * Storage transfer service or Cloud Data Transfer Service - allows you to quickly import ONLINE data into cloud storage
        * Can setup repeating schedule for transferring data 
        * Can be used to also transfer data within cloud storage - from one bucket to another
    * Transfer Appliance Service - is an OFFLINE secure high capacity storage server that you setup in your datacenter - you fill it with data and ship it to an ingest location where data is uploaded to GCS - choose for transferring On-Prem data

## Infra-Services
* `Pub/Sub`
    * Asynchronous meaning  - no need to maintain connection to client
    * Pubsub only stores messages for 7 days whist kafka is unlimited
    * It autoscales better than Kafka 
* `Cloud Composer`
    * GoogleCloudStorageToBigQueryOperator to move from GCS to BQ
    * Cloud Composer can be built to monitor workflows and detect say file condition and trigger something

## AI and ML
* `Cloud Vision API`
    * Only accepts base64-encoded strings
* `Cloud Speech-To-text`
    * Use Cloud Speech-to-Text API, and send requests in an synchronous mode for short audio files
    * native sample rate is recommended over resampling.
    * Synchronous speech recognition returns the recognized text for short audio (less than ~1 minute) in the response as soon as it is processed. To process a speech recognition request for long audio, use Asynchronous Speech Recognition.
* `AI Platform`
    * Monitor the status of the Jobs object for 'failed' job states.
* `Cloud Natural language `
    * Used to derive insights from unstructured text
* `Tensorflow/DNN`
    * Regularisation uses techniques to make model simpler and reduces overfitting

## Test Results
* GCP PDE Provided Practice Questions:
    * Attempt 1: 15/29 - 51.7% (Week 1)
    * Attempt 2: 21/29 - 72.4% (Week 5)
* Whizlabs Free test:
    * Attempt 1: 15/20 - 75.0% (Week 6)
