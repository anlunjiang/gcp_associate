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
* `Federated query` - where it can read and write and transform from an external data connection, gcs excel 
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


# Building a Data Lake

A data lake is a scalable, secure data platform that allows enterprises to ingest, store, process and analyse any type of volume or information

Imagine data lake being a central area where all the raw materials are for a building project analogy. THe raw materials will then be transformed (cut, melted ) into your data warehouse

* An orchestrator governs all aspects of this workflow - e.g. apache airflow runing in a cloud composer environment

GCS is a good option to be your data lake, but not the only option. bq can easily be both your datalake and your data warehouse. But data lake captures every aspect of your business operation. The data is stored in natural/raw format, usually as object blobs or files

## Data Lake
* Retains data in native format
* Supports all data types and all users
* Adapts to change easily
* Application specific

## Data Warehouse
* Loaded only after a use case is defined
* processed/organised/transformed
* provide faster insights
* current/histroical data 
* Consistent schema

### Choosing between EL, ELT and ETL
* EL - data is already in the same data type and format - data imported as is. If source and target already have same schema, then go for it
* ELT - Data loaded is not the final form. Can do when transformation complexity and volume is low. bq sql to transform raw data loaded into bq and write to a new table
* ETL - Transformation is essential and will reduce the data size - reducing the network bandwidth you need. e.g dataflow to transform

## Why cloud storage as data lake
* Data Persistence beyond vm and clusters
* File system compatibility - allows forward slashes as part of an object name - even though it is flat files
* Durable data storage 
* High/strong consistency  - all data is the same with lower availability
* Moderate latency and high throughput
* Data replication - heartbeat check to like hdfs. GCS replicates multiregional for multi regional buckets and same logic with single region buckets across zones
* Serves data from closes replica
* Has object lifecycle features where you can version or say to delete after 30 days or  move to another storage class nearline to coldline

Good to have buckets local to where you are processing - saving network charges. Note that bucket location cannot be changed. For maximum availability choose multi regional - so that even say if a region is flooded you can access your data as it's stored in different data centers

![confu](./Datalakes_warehouses/images/storage.PNG =500x)

### GCS controlling access
Two methods:
* Cloud IAM
    * Set at a bucket level - uniformly applies access rules to all objects, bucket reader, writer, owner roles etc
    * IAM also controls the ability to create access lists
* Access Lists
    * can be applied at the bucket level but also more fine grained: To individual objects

Encryption:  
* Data is encrypted by google managed keys - this is automatic and immutable - Data encryption Key DEK
* The data encryption key itself is then also encrypted by Key encryption key - KEK
* You can also custom manage the keys yourself and even supply your own keys and rotation of key mechanism
* Client Side encryption - data is encrypted before uploaded - is possible - DEK and KEK still works without knowing if the object
  its encrypting is already encrypted once
* Data access is logged
* Can put a lock on an object when auditing - or holds - so the object you want to audit does not change, this can be applied to the bucket itself as well

* share signed url that expires after an amount of time

## Other options for data lake in GCP
GCS is good but not for transactional workloads - latency is low but not low enough to support high freq writes. Cloud SQL or Firestore would work better - depending if you want RDBMS or nosql. 

* OLAP - Online Analytical Processing - generally scanning lots of data for decision making, complex queries - mostly read data - not good for datalake, more on data warehouses on curated data
* OLTP - Online Transactional Processing - Short simple queries relating to small amount of read and write - very fast, mostly write data

* Data engineers build the pipelines between OLTP and OLAP

Cloud SQL is good for most use cases -  but if you want global distribution - then use cloud spanner - which handles updates in different geographical locations. Also if your data is too large to fit into a cloud sql instance. Spanner is distributed and scalable. 

* However if not huge data loads then cloud sql is more cost effective for OLTP
* and big query for OLAP  - or bigtable if you need ultra low latency (milliseconds ) or extreme through put (millions of rows/second) 


# CLoud SQL

Fully managed rdbms - supports mysql, postgres and sql server without the db  admin jargon. 
* Also accessible by other GCP services like app engine or compute engine and other guis that devs are used to
* Encrypted, managed backups up to 7 per instance
* vertical scaling for read and write
* horizontal scaling for just read
* replication and fail replica as well on different zones. There is a master instance and replicas, if the master fails the replica becomes the master and a new replica is made
    * can also offload read requests to replica to ease workload of master

* Fully managed - 
    * No setup, automated backups, replicated and highly available
    * but you can still ssh into it and still control the hardware
    * data proc - can help you setup hadoop and spark workloads without setup - but you can ssh in
* Serverless - 
    * Next step up, no server management, fully managed security, pay for only usage
    * treat more like an api
    * pubsub, bq is serverless, dataflow, gcs is also technically serverless - since you dont interact with the hardware

> # Modernising Data Lakes and Data warehouses

Main difference between the two is that a data lake is just raw data and data warehouse brings the data together within a schema - available for querying and data processing. 

Data warehouses should be:
* Optimised for access and high speed query
    * Distributed parallel processing 10 seconds but actually SLOT TIME CONSUMED can be 2.5 hours - great speed!
        * A bq slot is just a combination of cpu memory and networking resources - slots alloted to a query depends on the complexity and project quota
* Scalable to terabytes or even petabytes
* Serverless and no-ops 
* Ecosystem of visualisation and reporting tools 
* Ecosystem of ETL and data processing tools 
* Up to the minute data - streaming capable - not just batch 
* Support Machine Learning without moving data out of the warehouse
* Security and Collaboration

These are all fitted by BigQuery which has these attributes that make it an ideal data warehouse 

## BigQuery
bq is serverless (remember not fully managed - since fm means you can still interact with the hardware)

### why is bq fast?
* Aside from massive parallel processing, bq tables are column oriented, so aggregation queries and selecting particular columns are fast.
* The data is distributed stored in a redundant way separate from compute clusters

### Features

* Structure: Project.dataset.table (dataset is like database in sql server). THe project is where the billing is associated with.
* Control: IAM is set through to the dataset level not the table or column
    * Views are possible for my fine grained control. Say we want to create a view in dataset B, which is a query of dataset A table - the two datasets must be in the same regionor multi region and cannot export data from the view
* Datasets like GCS buckets can be multiregional or regional
* Schema can be manually entered or supplied through a json file 
* Also like GCS data is encrypted with DEK data encryption keys  - also can supply your own keys (customer managed keys)
* Logs are immutable and logged to stackdriver for audit purposes
* Federated Queries: query datasets in different locations such as gcs
* Cached results for 24hours in a temp table so if you run the same query again within 24hours you will not be charged twice 
* Query validator to estimate costs of a query
* Separate cost of storage and cost of queries
    * Project A hosting dataset A can allow project B to query dataset A - and the cost will go to project B
* bq offers 1TB of querying for free everymonth
* Data Transfer service can copy large datasets from different project to yours inseconds

## BigQuery DataLoading

Bigquery supports loading files of different formats, csv, avro, parquet, orc etc - gzipped files are supported but is slower.
* Load jobs are asynchronous - no need to maintain connection to client and dont affect other bq resources

* Avro formats: Self described schema - bq can determine the schema directly. 
* JSON or CSV: bq can autodetect the schema - but manual verification is recommended 
    * Schema can be explicitly parsed in whilst running the command, skip leading rows flag can skip header rows 
* Cloud function can listen to new files arriving to say GCS and load the jobs to bq automatically 
* APi is available to import data with other resources compute engine
* bq addresses DR and backups by creating a point in time snapshot up to 7 days for recovery
    * This is directly within the sql query where you can select a table from a point in time `FOR SYSTEM_TIME AS OF TIMESTAMP`

### BigQuery Data Transfer Service
Builds and manages data warehouse for extract Load.  - It is a managed service without coding
* Can transfer from SaS and across regions 
    * Buckets are not needed 
* Backfill compatible  with scheduling once a day 

### BigQuery temp functions
UDF's are available in SQL JS and scripting. SQL is recommended since bq can optimise the query better. Stored procs, scripts and permanent saved functions are available that you can share 

## Metadata

like how sql server has information schema schema for metadata about a database instance. BigQuery can have similar features

a bq dataset has dunder commands - you can select * from dataset.__TABLES__ to review the tables within a dataset, like row size, size, creation time etc

Bigtable also has INFORMATION SCHEMA! select * from dataset.information_schema.columns. Unlike in sql server where the info schema is bound to an instance, Each dataset in bq has its own information schema for metadata capture (partitioning columns, sh)

## Schema Design 

| Key                | Normalization                                                                                                                  | Denormalization
|--------------------|--------------------------------------------------------------------------------------------------------------------------------|----------------
| Implementation     | Normalization is used to remove redundant data from the database and to store non-redundant and consistent data into it.       | Denormalization is used to | combine multiple table data into one so that it can be queried quickly.
| Focus              | Normalization mainly focuses on clearing the database from unused data and to reduce the data redundancy and inconsistency.    | Denormalization on the other hand focus on to achieve the faster execution of the queries through introducing redundancy.
| Number of Tables   | During Normalization as data is reduced so a number of tables are deleted from the database hence tables are lesser in number. | On another hand during Denormalization data is integrated into the same database and hence a number of tables to store that data increases in number.
| Memory consumption | Normalization uses optimized memory and hence faster in performance.                                                           | On the other hand, Denormalization introduces some sort of wastage of memory.
| Data integrity     | Normalization maintains data integrity i.e. any addition or deletion of data from the table will not create any mismatch in the relationship of the tables. | Denormalization does not maintain any data integrity.
| Where to use       | Normalization is generally used where number of insert/update/delete operations are performed and joins of those tables are not expensive. | On the other hand Denormalization is used where joins are expensive and frequent query is executed on the tables.

Denorm for when joins are frequent and expensive - norm when transactional and integrity is essential

Bigquery generally performs better with denormalised data - however Grouping on a 1-to-many field in flattened data can cause shuffling of data over the network

![confu](./Datalakes_warehouses/images/norm_shuffle.PNG =500x)

Big query can handle this by using nested and repeated columns to improve efficiency

![confu](./Datalakes_warehouses/images/nestedbq.PNG =500x)

Different levels of granularity can now be supported within a single denomralised table - in structs - prejoined tables within a table. You can nest these structs within your table. all arrays in a struct are the same granularity. This is good to work with the columnar stored based format of bq

Arrays have the MODE to be repeated instead of nullable (arrays are not nullable - greyed out)
Structs have type RECORD - each struct has nested columns as part of it - one of which can be an array which is repeated. Array are a column type meaning that they are repeated within a single row 

* THis really reduces the amount of shuffling and efficiency

```sql
select 
    block_id, 
    max(i.input_sequence_number) as max_seq_numner,
    count(t.transaction_id) as num_txn_per_block,
from `bigquery-public-data.bitcoin_blockchain_blocks` as b
, b.transactions as t
, t.inputs as i -- comma is shorthand in bq for cross join
group by block_id;
```

for nested structs - you need t unpack them by using the UNNEST sql method so bq can see the rows

## Optimising with Partitioning and Clustering

Reduce the cost and amount of data read by partitioning - reduces read when having a where clause, this plus if you dont query select * then you only read a tiny fraction of the dataset 

![confu](./Datalakes_warehouses/images/partition_bq.PNG =500x)

If also improves query cost performance and reducing data queried 

Bigquery automically sorts the data based on values in the clustering columns for better filter performance and aggregations 

You can partition and cluster on table creation time: 
PARTITION BY DATE  
CLUSTER BY ID

Partition on
* TIme pseudocolumn
* time user datetime column
* int

YOu can also expire partitions more than x days old

Over time as more data gets written to a table - you may need to resort the table - you can recluster by select * and rewrite to the table - DML can force the recluster

* With Bigquery though - it now reclusters for you periodically  - it is free and autonomous

* When to use clustering
    * Your data is already partitioned on a DATE or TIMESTAMP or Integer Range
    * You commonly use filters or aggregation against particular columns in your queries.
    * clustering is a bit like indexing in sql rdbms
# Data Processing

Cloud Dataproc and Dataflow can process data in an intermediate system before loaded to a data warehouse


