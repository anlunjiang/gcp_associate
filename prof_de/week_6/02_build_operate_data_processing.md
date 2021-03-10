# Building and Operationalising Data Processing Systems

![choose_store](./images/choose_datastore.PNG =800x)

* Cloud Storage
    * Signed URLS to temporarily give access to developers/users - auto expires
    * Lifecycles - delete after a certain amount of time
    * storage class:  is standard, nearline, coldline, archive
* Cloud SQL
    * MySQL and MSSQL
    * Can configure read replicas to cater for larger throughput
* Cloud BigTable
    * High throughput and millisecond latency
    * Columnar store and uses cbt
* Cloud Spanner
    * Fully Managed RDBMS with transactional consistency
    * Strongly typed: schema must be defined and data types for each col
    * Seocndary Indexes
    * Horizontally scalable - global
* Cloud DataStore
    * NoSQL database
    * ACID compliant
    * Kind category with containers for property data

![choose_store](./images/dataoptions.PNG =800x)

* Pub/Sub
    * Stream from PubSub to bigquery or bigtable with unbounded datasets. 
        * Bigtable for higher throughput
    * Guarantees delivery but NOT order of messages. At least once means that repeated delivery of the same message is possible if the subscribers are not responding.
        * Dataflow processing can remove dupes and use internal pub/sub timestamp to work out out of order messages
    