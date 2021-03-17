# Weakpoints and Technology Summaries

## Processing
* `Compute Engine`
    * You can do easy snapshot restoration to a persistent disk  AS WELL AS restoring a snapshot to create a new VM Instance directly
* `Dataproc`
    * Runs on Spark cluster to process
    * Create separate Dataproc clusters for different purposes - since compute is separate to storage 
    * Google recommends Cloud Storage instead of HDFS as it is much more cost effective especially when jobs aren’t running.
    * Separation of compute and storage
    * If a Dataproc job fails when writing to GCS - any temp files wont be deleted and will need to be removed manually
        * Successful jobs will automatically remove temp files 
    * Situation to continue to use HDFS is if you have heavy IO workloads - partitioned writes
        * Modify HDFS data frequently or you rename dirs
    * You can have custom installs to cluster lke zeppelin and install using initalisation actions from yaml files
        * you can also specify custom dataproc image if all jobs require the same requirements - quicker than initalisation actions
    * Avoid iterating sequentially over many nested directories - since GCS is only simulating a file structure
    * To access web UI - Using a SOCKS proxy may be preferable to using local port forwarding since the proxy:
        * You can also use normal port forwardignvia firewall rules but socks is preferred
    * You can have pre-emptible nodes - but not only them. A Dataproc cluster must have at least 2 stardard worker nodes 
    * You can isolate a cluster from the public by using the --no-address flag to prevent public IP. Make sure to enable Private Google Access though
    continue to access APIs
* `Dataflow` 
    * Uses Apache beam to process 
    * You can use Dataflow to write to bq and bt. Bigtable and BigQueryIO transforms can be used in the Pipeline
    * Use Combine instead of GroupBy to avoid shuffling
        * Groupby only uses one worker per key creating bottlenecks- unlike combine distributes keys across multiple workers
    * Dataflow templates enable non devs to create pipelines
    * Use drain option tells dataflow to finish up existing jobs and not start new ones
    * THere is no data sharing option between pipelines - youd need to have an io manager to write to an external source like GCS
    * Side inputs inject addtional runtime data
    * PCollections - batch or stream - distributed
        * Immutable - any changes creates a new PCollection - the old one is unchanged 
        * Bounded PCollection - data has a fixed size  - NOT that the PCollection size is limited
        * Unbounded - data is not fixed in size
        * Dont have to be read from extenral source - can create a PCollection from memory
        * Serialisation is done automatically into and from byte strings
    * PTransforms using Pipe symbol to join
        * Handles IO and transformations
    * Flatten - Works like a SQL Union - it unions many Pcollections that store the SAME DATATYPE into one large Pcollection 
    * Partition - Splits PCollections into smaller PCollections. e.g. split by quartile and 1 quartile of data requires different processing that others 
    * Dynamic rebalancing - if nodes are taking a long time and others have finished - the remaining work is auto rescaled - savings time and money 
    * ParDo parallel do -  acts on one item at a time in the PCollection  
        * Multiple instances of class on many machines and does not contain state; for filtering datasets, formatting, computing etc
        * DoFn must be idempotent, threadsafe and serialisable
    * 3 Kinds of windows fit most circumstances:
        * Fixed - divides data into time-based finite chunks, e.g. day, month - consistent non overlapping intervals
        * Sliding - required when doing aggs over unbounded data - can overlap - e.g. running average 
        * Sessions - defined by minimum gap duration - timing triggered by another element - good for communication in bursts , web sessions from a user
    * Handling latency and windowing:
        * System will calculate lag by taking expected arrival time difference against actual arrival time 
        * Dataflow keeps track on lag with Watermarking: Provides flexibility for a little lag time - the watermark time is the limit cutoff for a window 
        * Late data can be discarded by default, or you can reprocess the window based off the late arrivals for java only
    * You can add triggers to happen after a watermark, processing time or count

## Pre-Post Processing
* `DataFusion`
    * Create a Spark or MapR for data ingestion from one source to another one
    * Can create tables directly from bq
* `DataPrep`
    * Create a Beam pipeline and run it on Dataflow
    * More designed for data preparation (as its name means), data cleaning, new column creation, splitting column
    * Has recipes that records your data edits
        * Once a recipe has been created you can schedule it to different destinations - different from manual runs
    * Cannot access DOC files
    * When available - it is easier to use dataprep suggestions instead of making recipes from scratch
    
## Storage
* `CloudSQL`
    * Can configure read replicas for reducing load of primary instances
        * Read replicas cannot be made highly available like primary instances. During a zonal outage,
         traffic to read replicas in that zone stops.
        * Once the zone becomes available again, any read replicas in the zone will resume replication from the primary instance
    * Can also use failover replica that will take over in the event of failure - switches to standby source
        * primary instance writes to a system db as a heartbeat - if missing heartbeat for 60s then failover is initiated
        * for High Availability
    * In the event of unsuccessful connections to database : exponential backoff retries
    * You can specify maintenance windows for planned downtime
* `BigTable`
    * By default, Cloud Bigtable is eventually consistent. To guarantee strong consistency you must limit queries to a single cluster in an instance by using an application profile.
    * An optimized Bigtable instance with a well-designed row key schema can theoretically support up to 10,000 write queries per second per node
    * Cloud BigTable CBT - The cbt tool is a command-line interface for performing several different operations on Cloud Bigtable, preferred over HBase
    * You can have bigtable production and development instances
    * Poor performance could be due to low nodes, hdd instead of ssh and rows in tables are large in data size - wide rows instead of tall and narrow
    * Once a BT instance is created - you cannot change the storage type (HDD/SSD), ID,  or downgrade from production to development (but can permanently upgrade)
        * You can however alter the number of cluster nodes, and number of clusters and display name - add nodes when 70% processing power is used
    * Have randomised IDs for row key for best performance - NOT sequential as that will lead to hotspotting
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
        * rows marked for deletion are not processed and will be removed in next compaction
    * Reverse timestamp if you read recent data regularly - physical data will be stored at the top which will be read first
    * Flexible schema - nosql
    * distribute read and writes evenly across row space of table for optimum performance
    * Works best with over 1TB of data
* `BigQuery`
    * Upon  joins, it is preferred to make large table on left and smaller table on right - for optimisation
    * AVRO is the preferred data type 
    * Datasets are immutable - cant change col name, data type, mode or delete col
    * Stucts and arrays
        * Arrays have MODE to be REPEATED instead of nullable (arrays are not nullable - greyed out)
        * Structs have type RECORD - each struct has nested columns as part of it - one of which can be an array which is repeated. 
        * To convert an ARRAY into a set of rows, also known as "flattening," use the UNNEST operator. 
            * UNNEST takes an ARRAY and returns a table with a single row for each element in the ARRAY.
            * UNNEST destroys the order of the ARRAY elements, you may wish to restore order to the table. 
                * To do so, use the optional WITH OFFSET clause to return an additional column with the offset for each array elemen
                * Then use the ORDER BY clause to order the rows by their offset.
    * Instead of using `COUNT(DISTINCT) ` use `APPROX_COUNT_DISTINCT` on large data streams when a small uncertainty is tolerable.
    * Bigquery supports wildcard syntax when querying from multiple table objects e.g. "`bigquery-public-data.noaa_gsod.gsod*`"
    * wildcard bigquery standard sql for multiple tables is `_TABLE_SUFFIX`
    * You can schedule query but NOT schedule exports for that youd need dataflow probably
    * Cache does not work if destination table is specified in job config
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
        * Logs are immutable and logged to stackdriver for audit purposes
    * bq addresses DR and backups by creating a point in time snapshot up to 7 days for recovery
        * This is directly within the sql query where you can select a table from a point in time 
    * Reclusters for you periodically  - it is free and autonomous
        * Your data is already partitioned on a DATE or TIMESTAMP or Integer Range
        * You commonly use filters or aggregation against particular columns in your queries.
        * clustering is a bit like indexing in sql rdbms
    * bq batch load is free but streaming is charged by size
    * add `warm_start = true` to your model options if you are retraining new data on an existing model for faster training times.
        * Note that you cannot change the feature columns (this would necessitate a new model).
    * Views are possible for my fine grained control. Say we want to create a view in dataset B, which is a query of dataset A table - the two datasets must be in the same region or multi region and cannot export data from the view
    * 1TB of free querying per month
    * Has support for GIS Geographical Information functions like ST_DISTANCE
    * You are charged as soon as you load data into BQ, there are two types of pricing - loading is free (except for streaming)
        * Active Storage Pricing - Pro-rated per MB/second
        * Long-term Storage Pricing - partitioned tables not edited 90+ consecutive days, pricing drops ~50% - only works if tables are partitioned
    * Soft Failure: Hardware is NOT destroyed - powerfailure etc - no data loss
        * In event of machine level failure - bigquery will continue running with only a few ms delay
    * Hard Failure: Hardware is destroyed
    * Bigquery multiregion: data stored in a region is backup in a separate region eu/us to provide resilience
        * Hard regional failure could lose recent data; the data not yet backed up offsite to a different region
        * Since backups are 1hour stale
* `Data studio` for visualising dashboards from bq
    * Compatible with YouTube as a data source!
    * Use dimensions if they are needed to be displayed in full - metrics is what you want to measure
    * Gets data from cache when possible. Disable caching from the report settings if you need latest data 
    * Max caching period is 12 hours
    * Two types of caching:
        * Responsive caching - report remembers requested data returned. Will look into this first
        * Predictive caching - analysis the dimensions/metrics and tries to predict query
            * This type of caching is only active for data sources that use data owner credentials to access data
    * You can add custom sql queries straight within data studio
* `Cloud Spanner`
    * Has secondary indexes - best created when loading data or creating the table
    * Do not use monotonically increasing primay keys
    * version 4 uuid is recommended to primary keys - universally unique identifier
* `DataStore`
    * NoSQL database
    * Data are properties contained in an entity in kind categories
    * Key word to look out for is "semi-structured" or "flexible" schema - since RDBMS are strict
    * scalable and avaiable
    * Handles sharding and replication
    * atomic - acid transactions - ACID Compliant
    * NOT built for huge volumes - less than 1 TB
    * to export  entitiies and data using gcloud datastore export command to export all entities in a database --async to prevent waiting
    * For entities where there is only one possible value - you can just not index these to avoid explosive indexing
* `Firestore`
    * NoSQL database with auto scaling
    * Used mainly for mobile applications
* `Storage Transfer Services`
    * Storage transfer service or Cloud Data Transfer Service - allows you to quickly import ONLINE data into cloud storage
        * Can setup repeating schedule for transferring data 
        * Can be used to also transfer data within cloud storage - from one bucket to another
        * Only to and from cloud storage- NOT say from cloud sql to bigquery directly
        * Works from other providors as well AWS and Azure
* `Transfer Appliance Service`
    * OFFLINE secure high capacity storage server that you setup in your datacenter - you fill it with data and ship it to an ingest location where data is uploaded to GCS - choose for transferring On-Prem data
* `Google Cloud Storage`
    * There a four storage classes:
        * Standard Storage - hot data
        * Nearline - once a month or less
        * Coldline - low cost once a quarter
        * Archive - dr
    * Bucket ids and gcp projects have to be globally unique 
    * Can have lifecycles to deelte objects after a certain period fo time
    * Signed URLS that wil expire after a time

## Infra-Services
* `Pub/Sub`
    * Asynchronous meaning  - no need to maintain connection to client
    * Pubsub only stores messages for 7 days whist kafka is unlimited
    * It autoscales better than Kafka  
    * serverless and provides infra to provide scalability
    * End to end encryption in transit and at rest
    * Builtin buffer for especially high loads and traffic spikes
        * However this is wasteful
    * can be batch or stream (batch is higher throughput but stream lower latency)
    * Delivers at least once - and will retry if no ack is received - dataflow can be used to dedupe by picking same message id and order by ts
    * A snapshot on the subscription is the easiest way to safeguard against application deployments by providing point-in-time recovery.
        * If the previous version of the application needs to be re-deployed, the subscription can be rolled-back to the point in time of the snapshot, and all subsequent messages will be re-processed.
    * Order of message can be done by attaching timestamp to each message
* `Cloud Composer`
    * GoogleCloudStorageToBigQueryOperator to move from GCS to BQ
    * Cloud Composer can be built to monitor workflows and detect say file condition and trigger something
    * DummyOperator for start and end for better visual representaion
    * DAG - A Directed Acyclic Graph is a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies.
    * Operator - The description of a single task, it is usually atomic. For example, the BashOperator is used to execute bash command.
    * Task - A parameterised instance of an Operator; a node in the DAG.
    * Task Instance - A specific run of a task; characterized as: a DAG, a Task, and a point in time. It has an indicative state: running, success, failed, skipped, ...
    * 2 types fo scheduling:
        * Pull - periodic - schedule normally
        * Push - Event driven - e.g cloud fucntion to watch fro gcs buket to have new file
* `Monitoring`
    * Stackdriver/monitoring plugin enables monitoring of on-prem and manually installed MySQL databases
        * Detects instance named mysql or check 3306 ports
        * Need to create a user to use the plugin that can run show_status
        * Install stackdriver plugin and configure fluentd plugin
    * Can monitor BigQuery and provide dashboards and charts for execution time, storage and slots allocated etc
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
* `Cloud IoT Core`
    * fully managed service and scalable to securely connect, manage, and ingest data from millions of globally dispersed devices
        * Secure by using asymmetric key pairs
    * Uses Pub/Sub underneath to aggregate dispersed device data
    * two main components:
        * A device manager for registering devices with the service, so you can then monitor and configure them.
        * A protocol bridge that supports MQTT (message queue telemetry transport) and HTTP, which devices can use to connect to Google Cloud.
    * Can have real time metrics with stackdriver 

## AI and ML
* `Cloud Vision API`
    * Only accepts base64-encoded strings
    * Can also give you colour palette and composition information
    * Provides how likely a description is accurate
* `Cloud Speech-To-text`
    * Use Cloud Speech-to-Text API, and send requests in an synchronous mode for short audio files
    * native sample rate is recommended over resampling.
    * Synchronous speech recognition returns the recognized text for short audio (less than ~1 minute) in the response as soon as it is processed. To process a speech recognition request for long audio, use Asynchronous Speech Recognition.
* `Cloud Natural Language API`
    * Lets you extract entities from text, perform sentiment and syntactic analysis, and classify text into categories
    * Entity and syntax analysis - can identify entities and label by types - e.g. person, location
    * Used to derive insights from unstructured text
* `AI Platform`
    * Monitor the status of the Jobs object for 'failed' job states.
    * Deploy Tensorflow models
    * Datalab supports JS, Py and SQL but NOT Java
    * Every staff member using their own Datalab is the correct method of sharing work
        * You can do version control by synchronising changes to the shared Cloud Source Repository.
* `Cloud VIdeo Intelligence`
    * Makes videos searchable and discoverable by extracting metadata with TEST API - anotates videos
* `Tensorflow/DNN`
    * Regularisation uses techniques to make model simpler and reduces overfitting    
    * Use TPU machine type Tensor Processing Unit - custom designed machine. 1 Cloud TPU  is 27 times faster whilst at 38% less cost of 8 v100 GPUs
    * Precision - Use to check accuracy when most outputs are positives TP/TP+FP
    * Recall - Use to check accuracy when most outputs are negatives - e.g. cancer diagnoses TP/TP+FN = 1 means cancer diagnoses escaping which is good
    * HyperParameter Tuning - Govern training process itself - e.g. how many hidden layers, nodes in hidden layers - not related to training data, usually constant during training, so weights are no hyperparameters - since they are learnt
    * When training you can specify master and worker type, worker count and paramterServerCount and type - you do this by selecting CUSTOM scaling tier
    * Has hierarchy of toolkit
        * High level API tf.estimator - train , evaluate, predict export
        * Componenets for custom NN models - tf.layers
        * Python API for full control
        * C++ API for low level
    * Tensorflow is lazy processing by default - but can change that with eager mode for dev work and experimentation 
    * RMSE and MSE - RMSE is in the units of measurements so its easier for context 
    * Cross-entropy - is a classification problem
    * Overfitting - model is too good - increase training data, increase regularisation and dropouts and decrease features
    * Tensorboard used to monitor training in tensorflow to look at properties of model (loss, model)
    * You can also train on TensorFlow and then predict on BigQuery - by changing `model_type` as tensorflow
    * L1 regularisation can reduces weights of less important features to zero/nearzero
    * Deep and wide model and ideal for recommendations
    * Wide models for memorisation
    * Deep model for generalisation
* `AutoML`
    * Quickly builds model for you - you can start with your own training set and use GCP models
    * Improve quality of subpar models:
        * AutoML Vision allows you to sort images by how "confused" the model is - by true/predicted label. Good to look through and make sure labels are applied correctly
        * Can add different images of different types, high res, low res
        * Consider removing labels altogether if you don't have enough training images.
    * AutoML will automatically stop training is the model isnt getting better
    * AUtoML NLP models are deleted if unused for 60 days up to 6 months
        * Must be periodically retrained as improvements are constant so retraining is ideal for better performance
        * For AutoML NLP, ensure you provide at least 10 documents per label, but ideally 100x more documents for most common lable than for least
    * AutoML Tables exist to perform ML on tabular data - not open source 
* `Kubeflow`
    * Packages multistep ML pipelines for kubernetes - benefits are like for cloud composer
    * Detailed UI for visualisign model and training details
    * AI Hub can be used to see existing AI pipelines

## Misc
* Fully managed - 
    * No setup, automated backups, replicated and highly available
    * but you can still ssh into it and still control the hardware
    * data proc - can help you setup hadoop and spark workloads without setup - but you can ssh in
* Serverless - 
    * Next step up, no server management, fully managed security, pay for only usage
    * treat more like an api
    * pubsub, bq is serverless, dataflow, gcs is also technically serverless - since you dont interact with the hardware
* Data lake is just raw data and data warehouse brings the data together within a schema - available for querying and data processing. 
* Denorm for when joins are frequent and expensive - norm when transactional and integrity is essential
* Load techniques
    * EL - data is already in the same data type and format - data imported as is. If source and target already have same schema, then go for it
    * ELT - Data loaded is not the final form. Can do when transformation complexity and volume is low. bq sql to transform raw data loaded into bq and write to a new table
        * Experimental datasets where you are not yet sure what kind of transformations are needed to make the data useful
    * ETL - Transformation is essential and will reduce the data size - reducing the network bandwidth you need. e.g dataflow to transform
* OLAP - Online Analytical Processing - generally scanning lots of data for decision making, complex queries - mostly read data - not good for datalake, more on data warehouses on curated data
* OLTP - Online Transactional Processing - Short simple queries relating to small amount of read and write - very fast, mostly write data
* AVRO serialises data and deserialisation using JSON for defining data types to serialise in binary - used for Hadoop HDFS
* For storage - only cloud SQL and bigtable are fully managed, the rest are serverless
* 3 types of failures:
    * Machine-level - hardware failure for only a single machine 
    * Zonal: rendering a single zone unavilable - .e.g power cut internet loss
    * Regional: entire region is affected , natural disasters
* Best practices is to maintain separate projects in an org for each env, dev, test, prod etc
## Test Results
* GCP PDE Provided Practice Questions:
    * Attempt 1: 15/29 - 51.7% (Week 1)
    * Attempt 2: 21/29 - 72.4% (Week 5)
    * Attempt 3: 29/29 - 100%  (Week 6)
* Whizlabs Free test:
    * Attempt 1: 15/20 - 75.0% (Week 6)
    * Attempt 2: 20.20 - 100%  (Week 6)
* Whizlabs Test 1:
    * Attempt 1: 38/50 - 76.0% (Week 6)
    * Attempt 2: 47/50 - 94.0% (Week 7)
* Whizlabs Test 2:
    * Attempt 1: 35/50 - 70.0% (Week 6) - Bad questions are ambiguous, could have scored higher, also v.tired
    * Attempt 2: 45/50 - 90.0% (Week 7)
* Whizlabs Test 3:
    * Attempt 1: 43/50 - 86.0% (Week 6) - +3 - some questions were bad shoudl be 90%
* Linux Academy:
    * Attempt 1: 41/50 - 82.0% (Week 7) 
    * Attempt 2: 40/50 - 80.0% (Week 7)