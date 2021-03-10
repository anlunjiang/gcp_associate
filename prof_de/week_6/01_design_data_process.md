# Preparing for the GCP PDE Exam

## Designing Data Processing Systems

* Review of DE on GCP:

###  Storage and Databases

* Managed Services - where you can see the individual instance or cluster - interact with infra
    * Cloud SQL - Transactional - not for unstructured data
    * Cloud BigTable - Ingest and Streaming high throughput data
* Serverless - no management of servers at all
    * Cloud Storage
    * Firestore - NoSQL document database with auto scaling - for mobile app development
    * Cloud Spanner - RDBMS that horizontally scales
    * DataStore - NoSQL - where data is property contained in an entity and in a kind category
    * BigQuery - 3 replica data, structs (RECORD type) and arrays (Repeated MODE)

* AVRO serialises data and deserialisation using JSON for defining data types to serialise in binary - used for Hadoop HDFS
* BigQuery Struct - Structs have type RECORD - each struct has nested columns as part of it - one of which can be an array which is repeated - reduces shuffling
* BigQuery Arrays  - MODE to be repeated instead of nullable: Array are a column type meaning that they are repeated within a single row 
* Storate Transfer Service - for transferring data from another cloud storage provider 


### Server base processing

* Compute Engine
* App Engine
* K8s Engine
* Cloud Functions

* Managed
    * Dataproc - Hadoop/Spark Workloads RDD data abstraction
        * spearation of compute and storage
* Serverless
    * Dataflow - Apache beam workloads - PCollection and PTransform pipeline abstraction -immutable
        * Python or Java compatible 
        * Combine is more efficient than GroupBy 
        * Clusters are managed automatically - autoscale
        * Works for both batch and stream
        * Fixed windows for bounded data
        * Side Inputs which allow different transformation within the same pipeline
        * ParDo - Parallel do - instantiation of a class process on each node - stateless
    * Bigquery  - Table query abstractions
        * Federated Queries to query straight from source without ingestion - e.g. GCS  CSVs
        * bq tool can upload data and run queries from cloud shell sdk


### AI

* Build your own:
    * AI Platform
    * Cloud TPU
    * AutoML
* Pre-Built:
    * Vision API - base64 encoded files only 
    * Speech-To-Text API - natice sampling rate preferred - short audio files synchronous - asynchronous for longer files
    * Video Intelligence API
    * Cloud Natural Language API
    * Cloud Translation API

### Pre-post processing services

* Transfer Appliance
* Dataprep - Creates a Beam pipeline and runs on DataFlow - data preparation and cleaning
* AI Platform Notebooks - JupyterLab notebooks
* Google DataStudio - for exploring bigquery data 
* Data Fusion - Creates a Spark pipeline and run it on Dataproc cluster - designed for data ingestion from one source to another
* DialogFlow


### Infrastructure Services

* Pub/Sub - no-ops serverless global message queue
    * Asynchronous - publisher never waits and subscriber can hold messages for 7 days
    * Often use PS to Dataflow that will dedupe and order - and then to bq for analysis
* Cloud Composer
* Cloud APIs
* Cloud monitoring (Previously Stackdriver)





