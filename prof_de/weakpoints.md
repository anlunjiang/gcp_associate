*  You are building storage for files for a data pipeline on Google Cloud. You want to support JSON files. The schema of these files will occasionally change. Your analyst teams will use running aggregate ANSI SQL queries on this data. What should you do?
    * Use BigQuery for storage. Select "Automatically detect" in the Schema section.
* You are designing a streaming pipeline for ingesting player interaction data for a mobile game. You want the pipeline to handle out-of-order data delayed up to 15 minutes on a per-player basis and exponential growth in global users. What should you do?
    * Design a Dataflow streaming pipeline with session windowing and a minimum gap duration of 15 minutes. Use "individual player" as the key. Use Pub/Sub as a message bus for ingestion.
    * A Is correct because the question requires delay be handled on a per-player basis and session windowing will do that. Pub/Sub handles the need to scale exponentially with traffic coming from around the globe. B Is not correct because Apache Kafka will not be able to handle an exponential growth in users globally as well as Pub/Sub.
* Your company is migrating their 30-node Apache Hadoop cluster to the cloud. They want to re-use Hadoop jobs they have already created and minimize the management of the cluster as much as possible. They also want to be able to persist data beyond the life of the cluster. What should you do?
    * Create a Dataproc cluster that uses the Cloud Storage connector.
    * A is not correct because the goal is to re-use their Hadoop jobs and MapReduce and/or Spark jobs cannot simply be moved to Dataflow. B is not correct because the goal is to persist the data beyond the life of the ephemeral clusters, and if HDFS is used as the primary attached storage mechanism, it will also disappear at the end of the cluster’s life.
* Your company’s on-premises Apache Hadoop servers are approaching end-of-life, and IT has decided to migrate the cluster to Dataproc. A like-for-like migration of the cluster would require 50 TB of Google Persistent Disk per node. The CIO is concerned about the cost of using that much block storage. You want to minimize the storage cost of the migration. What should you do?
    * Put the data into Cloud Storage.
    * correct because Google recommends using Cloud Storage instead of HDFS as it is much more cost effective especially when jobs aren’t running.
* You designed a database for patient records as a pilot project to cover a few hundred patients in three clinics. Your design used a single database table to represent all patients and their visits, and you used self-joins to generate reports. The server resource utilization was at 50%. Since then, the scope of the project has expanded. The database table must now store 100 times more patient records. You can no longer run the reports, because they either take too long or they encounter errors with insufficient compute resources. How should you adjust the database design?
    * Normalize the master patient-record table into the patients table and the visits table, and create other necessary tables to avoid self-join.
* You created a job which runs daily to import highly sensitive data from an on-premises location to Cloud Storage. You also set up a streaming data insert into Cloud Storage via a Kafka node that is running on a Compute Engine instance. You need to encrypt the data at rest and supply your own encryption key. Your key should not be stored in the Google Cloud. What should you do?
    * Supply your own encryption key, and reference it as part of your API service calls to encrypt your data in Cloud Storage and your Kafka node hosted on Compute Engine.


* Dataflow does not support Spark.
* to use custom Spark transforms; use Dataproc. ANSI SQL queries require the use of BigQuery.
* bigquery wildcard `` use backticks
* synchronous mode is recommended for short audio files.
* Build an application that calls the Cloud Vision API. Pass client image locations as base64-encoded strings.
* Use Cloud Audit Logs to review data access - this is the best way to get granular access to data showing which users are accessing which data.

20/30