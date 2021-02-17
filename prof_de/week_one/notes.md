# Data Lakes and Data Warehouses

[[TOC]]

Data Lake: Brings together data from across the enterprise into a single location - to say cloud storage bucket  - for easier accessibility
Considerations to take:
* Handle all the types of data types
* Scale to meet demand
* High throughput ingestion
* Access control?
* connectability?

GCS - is a good place to store raw data - similar to hdfs, easier to migrate as well. It supports many different data types and is connectable to bq

## ETL

You can extact from GCS using dataproc and dataflow, pubsub and dataflow can handle streaming as well for real time analytics

### Challenges
* Access to data
    * data is siloed in many upstream source systems - each dept can have different systems
* data quality and acc
    * data warehouse is usually curated data from data lake
* Availablility of compute resources
    * on prem workloads need to share resources and make sure they are used efficiently and not wasted
* Query performance
    * serverless data warehosue bq is a good way to manage

# Big Query

Google's Petabyte scale serverless data warehouse - it replaces the typical hardware setup - supporting datasets tied to gcp projects 
* This would be a good replacement to hive in hadoop and gsutil to replace hdfs
* You also dont need to provision resources before using bq
* also has gis geographical data 
* can query data lake directly - simplifying etl pipelines, can leave data in place and still join with other data sources at high performance

![confu](./Datalakes_warehouses/images/data_pipeline.PNG =500x)

Data warehouse considerations
* Serve as a sink for both batch and stream?
* scalable?
* how is the data organised, access controlled, catalogued
* performance
* maintenance


* bq has bi engine which means no more olap cubes or separate bi servers for dashboard performance. It natively integrates with bq for streaming real time refresh
* stackdriver - logging spending, resources, which are the most popular datasets, which queries were run, sensitive data?

# Cloud SQL
Cloud SQL is a fully managed sql server, postgres or mysql rdbms - transactional database

Why bq Aaaannndd cloud sql? rdbms are optimised for data from a single source and high throughput read/writes
* cloud sql only scales to tb, whilst big query, pb
* cloud sql is ideal for back end  data base apps, whilst bq can connect to many sources
* cloud sql is record based, bq is column based, more hybrid between sql and nosql

## Governance

> `Cloud data catalogue + data loss prevention api`  
metadata from all datasets in pubsub, bq, gcs available - think of it like collibra
* you can flag datasets with PII - and label them and see all datasets based on those labels
* DLP can easily classify sensitive and PI information -  it can also redact that data 

# Cloud composer 

good substitute to apache airflow - for scheduling and orchestration, triggers etc 


