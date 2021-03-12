# Weakpoints and Technology Summaries

## Processing
* `Dataproc`
    * Runs on Spark cluster to process
    * Create separate Dataproc clusters for different purposes - since compute is separate to storage 
    * Google recommends Cloud Storage instead of HDFS as it is much more cost effective especially when jobs aren’t running.
    * Separation of compute and storage
    * If a Dataproc job fails when writing to GCS - any temp files wont be deleted and will need to be removed manually
        * Successful jobs will automatically remove temp files 
* `Dataflow` 
    * Uses Apache beam to process 
    * You can use Dataflow to write to bq and bt. Bigtable and BigQueryIO transforms can be used in the Pipeline
    * Use Combine instead of GroupBy to avoid shuffling
        * Groupby only uses one worker per key creating bottlenecks- unlike combine distributes keys across multiple workers
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
* `CloudSQL`
    * Can configure read replicas for higher availability and throughput
    * Can also use failover replica that will take over in the event of failure - switches to standby source
        * primary instance writes to a system db as a heartbeat - if missing heartbeat for 60s then failover is initiated
* `BigTable`
    * Cloud BigTable CBT - The cbt tool is a command-line interface for performing several different operations on Cloud Bigtable, preferred over HBase
    * You can have bigtable production and development instances
    * Once a BT instance is created - you cannot change the storage type (HDD/SSD), ID,   or downgrade from production to development (but can permanently upgrade)
        * You can however alter the number of cluster nodes, and number of clusters and display name
    * Tall and Narrow tables for best performance
    * For testing:
        * Use production instance
        * Use at least 300GB of data to test
        * Before test run a heavy pre-test for several minutes
        * Run test for at least 10 minutes
        * Do not scale up instance just before test starts - needs time to balance data across nodes
    * Data is stored in colossus - using tablets as datastructures
        * tablet metadata is stored on nodes in bigtable cluster - repointing is easy and fast in the event of node failure
        * Thus separates the processing and storage
    * No secondary indexes
    * Learns usage and reorganises periodically to maximise speed - called compaction - at this stage rows marked for deletion are also deleted
    * Reverse timestamp if you read recent data regularly - physical data will be stored at the top which will be read first
    
* `BigQuery`
    * Datasets are immutable - cant change col name, data type, mode or delete col
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
    * Non-deterministic functions like CURRENT_TIMESTAMP() and NOW() has no cache
    * Bigquery has flat rate fees, but also can set custom query quotas for each user or the project
    * Default encoding is UTF-8, if you use another you can specify it when you import your data so bq can convert before import
    * You can now assign access on the table or view level
    * Always consider using data in place instead of ETL
    * Limit statement does not actually decrease the work needed for reading
    * Query explanation map: can diagnose performance issues
        * wait - orange
        * read - green
        * compute - red
        * write - blue
    * Cloud Monitoring to monitor usage
* `Data studio` for visualising dashboards from bq
    * Compatible with YouTube as a data source!
    * Use dimensions if they are needed to be displayed in full - metrics is what you want to measure
    * Gets data from cache when possible. Disable caching from the report settings if you need latest data 
* `Cloud Spanner`
    * Has secondary indexes
* `DataStore`
    * NoSQL database
    * Data are properties contained in an entity in kind categories
* `Storage Transfer Services`
    * Storage transfer service or Cloud Data Transfer Service - allows you to quickly import ONLINE data into cloud storage
        * Can setup repeating schedule for transferring data 
        * Can be used to also transfer data within cloud storage - from one bucket to another
* `Transfer Appliance Service`
    * OFFLINE secure high capacity storage server that you setup in your datacenter - you fill it with data and ship it to an ingest location where data is uploaded to GCS - choose for transferring On-Prem data

## Infra-Services
* `Pub/Sub`
    * Asynchronous meaning  - no need to maintain connection to client
    * Pubsub only stores messages for 7 days whist kafka is unlimited
    * It autoscales better than Kafka 
* `Cloud Composer`
    * GoogleCloudStorageToBigQueryOperator to move from GCS to BQ
    * Cloud Composer can be built to monitor workflows and detect say file condition and trigger something
* `Monitoring`
    * Stackdriver/monitoring plugin enables monitoring of on-prem and manually installed MySQL databases
        * Detects instance named mysql or check 3306 ports
        * Need to create a user to use the plugin that can run show_status
* `Cloud Scheduling`
    * CRON Job scheduler for scheduling simple stuff
* `IAM`
    * Admin is greater than Owner for services like BQ - highest is Admin
    * Resource Hierarchy - Organisation -> Folders -> Projects -> Resources
        * higher up policies that are in conflict with low level policies always win
    * Encryption:
        * Default - Encryption keys DEK are encrypted again with KEK key encryption keys
        * CMEK - Customer managed: uses Cloud Key Management Service - can create, revoke keys using Google generated keys
        * CSEK - Customer supplies own encryption keys - google has no recovery though if lost - NEVER STORED ON DISK
            * Key is purged once operation is complete

## AI and ML
* `Cloud Vision API`
    * Only accepts base64-encoded strings
    * Can also give you colour palette and composition information
* `Cloud Speech-To-text`
    * Use Cloud Speech-to-Text API, and send requests in an synchronous mode for short audio files
    * native sample rate is recommended over resampling.
    * Synchronous speech recognition returns the recognized text for short audio (less than ~1 minute) in the response as soon as it is processed. To process a speech recognition request for long audio, use Asynchronous Speech Recognition.
* `AI Platform`
    * Monitor the status of the Jobs object for 'failed' job states.
    * Deploy Tensorflow models
* `Cloud Natural language `
    * Used to derive insights from unstructured text
* `Tensorflow/DNN`
    * Regularisation uses techniques to make model simpler and reduces overfitting    
    * Use TPU machine type Tensor Processing Unit - custom designed machine. 1 Cloud TPU  is 27 times faster whilst at 38% less cost of 8 v100 GPUs
    * Precision - Use to check accuracy when most outputs are positives TP/TP+FP
    * Recall - Use to check accuracy when most outputs are negatives - e.g. cancer diagnoses TP/TP+FN = 1 means cancer diagnoses escaping which is good
    * HyperParameter Tuning - Govern training process itself - e.g. how many hidden layers, nodes in hidden layers - not related to training data, usually constant during training, so weights are no hyperparameters - since they are learnt
    * When training you can specify master and worker type, and worker count
    * Has hierarchy of toolkit
        * High level API tf.estimator
        * Componenets for custom NN models - tf.layers
        * Python API for full control
        * C++ API for low level
    * Tensorflow is lazy processing by default - but can change that with eager mode for dev work and experimentation 
    * RMSE and MSE - RMSE is in the units of measurements so its easier for context 
    * Cross-entropy - is a classification problem
    * Overfitting - model is too good - increase training data, increase regularisation and dropouts and decrease features
    * Tensorboard used to monitor training in tensorflow to look at properties of model (loss, model)
* `AutoML`
    * Quickly builds model for you - you can start with your own training set and use GCP models
    * Improve quality of subpar models:
        * AutoML Vision allows you to sort images by how "confused" the model is - by true/predicted label. Good to look through and make sure labels are applied correctly
        * Can add different images of different types, high res, low res
        * Consider removing labels altogether if you don't have enough training images.

## Test Results
* GCP PDE Provided Practice Questions:
    * Attempt 1: 15/29 - 51.7% (Week 1)
    * Attempt 2: 21/29 - 72.4% (Week 5)
* Whizlabs Free test:
    * Attempt 1: 15/20 - 75.0% (Week 6)
    * Attempt 2: 20.20 - 100%  (Week 6)
* Whizlabs Test 1:
    * Attempt 1: 38/50 - 76.0% (Week6)
