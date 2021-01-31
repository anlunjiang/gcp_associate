# Comparing Storage Options

 [[TOC]]

# Summary 

| Cloud Storage                 | BigTable             | BigQuery                      |
|-------------------------------|----------------------|-------------------------------|
| Durable                       | Sparsely Populated   | Structured analytical         |
| Object Store (Non Structured) | Structured Data      | Pay for what you use          |
| Highly Available              | Low Latency / Updates| Calculate bytes read --dryrun |
| -                             | NoSQL database       | Charged for data read only    | 
| -                             | Billions (PB) of rows| -                             |


| Cloud SQL                         | Cloud Spanner               | Cloud Firestore                       | Cloud Datastore
| ----------------------------------| ----------------------------|---------------------------------------|-------------------
| Structured Data                   | Global and Consistent       | NoSQL                                 | NoSQL
| Relational Database               | Relational                  | Mobile SDK interactions with db       | None Relational
| Cannot scale horizontally well    | Replicated                  | None Relational Database              | Built on top of BigTable
| -                                 | Transactions                | -                                     | SQL Syntax
| -                                 | Fast access                 | -                                     | Transactions

# Cloud Storage

Cloud storage is highly durable and available non structured object store. 
* An object would consist of a file and some metadata about that file - each object has two parts
    * Metadata is a list of key-value pairs
* Objects are immutable, they cannot be changed  - no update, append, truncate
    * However they can be overwritten 
* They are stored in buckets - similar to folders, but flat structured in reality
* They can host static websites - good for marketing, signup forms 

## Storage Classes
* There are 4 different storage classes
    * Multi-Regional 
        * Data accessed frequently, video, media
    * Regional
        * Data accessed frequently within a region, data analytics, data processing
    * Nearline
        * Data accessed less than once a month, backup
    * Coldline
        * Data accessed less than once a year, backups, DR

## Signed URLs

Cloud storage can produce signed URLS that are secure and with an expiry date - this way you can access the data forever

# Big Table
 A sparsely populated table for structured analytical data with low latency and updates
 * Empty columns do not consume any space
 * Uses a row key to look up data - sotred in columns


## Scalability
Designed for massive amounts of data, petabytes
* A cluster is created with at least 3 nodes that all run bigTable
    * This cluster is referred to as an instance, these nodes are always running so is more expensive
* data in one cluster will be replicated making a master master replication
    * eventual consistency

# BigQuery
Fully managed Data Warehouse using SQL like syntax
* Pay for what you actually use:
    * streaming updates (Billing exports)
    * Queries you run BASED ON DATA READ
* You can calculate cost of the query by calculating the bytes to be read
    * CLI --dryrun returns number of byutes read
    * REST API can set --dryrun parameter to be true
    * UI Query validator
    * GCP Price calculator can then be used to figure out the cost 

# Cloud SQL

Managed database service that supports mySQL and PostgresSQL
* Relational and Structured
* Easy to migrate from on-prem to cloud 
* Tailored for high availability and replication, but not mass horizontal scaling

# Cloud Spanner 
Combine best aspects of NoSQl and RDBMS
It is a Globally Dsitributed Strongly Consistent database
* Horizontally scalable and relational database service
* Uses ANSI SQL
Good for applications that need SQL databases but need horizontal scaling

* Instances are created that are always running 

# Cloud Firestore

NoSQL datastore option
* Great for mobile applications with the modile SDK - mobile device interacting with database

# Cloud Datastore 
Non Relational NoSQL datastore - interact with backend code
* For Unstructure data  
* Fully Managed noSQL database on top of BigTable
* Great for web and mobile apps with simple querues (catalogs)

# Flow Chart Options

![module_hierarchy](../../images/storage_options.png =800x)
