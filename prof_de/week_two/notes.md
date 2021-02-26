# Building Batch Data Pipelines on GCP
EL, ELT and ETL

When to use EL:
* Batch load of historical data
* Scheduled periodic loads of log files (e.g. once per day)
* From GCS say to bq native

When to use ELT:
* Experimental datasets where you are not yet sure what kind of transformations are needed to make the data useful
* Production dataset where the transformation can be expressed in SQL easily 
* Transform data on the fly using bq views or store into new tables from GCS

When to use ETL:
* transformations needed are too complex for sql/bigquery
* raw data needs to be quality controlled or enriched
* data loading is happening continuously
* extract data from ps, gcs etc and transform using dataflow (say running beam supports both batch and stream) and write to bq

Common reasons for data quality processing can be 
* Validity - our of range, empty fields, corrupted data
    * bq views to filter rows and aggregations using WHERE and HAVING. remove nulls and blanks
* Accuracy - lookup datasets do not exist
    * test data against known good values - in() subquery 
* Completeness - Missing Data
    * nullif, coalesce, union and join for enriching data
    * check hashsum md5 to check data integrity 
* Consistency - Duplicates, concurrency issues
    * select distinct from bq, or group by
* Uniformity - Same unit of measurement
    * bq format function


For ETL it is recommended to use Dataflow and BigQuery unless you have specific needs:

| Issue                    | Solution             |
|--------------------------|----------------------|
| Latency, throughput      | Dataflow to Bigtable | 
| Reusing Spark pipelines  | Dataproc             |
| Visual Pipeline building | Data Fusion          |

Dataproc is a maanged service for batch processing, querying, streaming and ML. Managed - so you can still interact with the hardware if you wish - but without the typical hadoop maintenance. 

Data Fusion is a fully managed, cloud native enterprise data, integration service for quickly building and managing data pipelines - , psst, its like alteryx as it has visual pipelines. 

You can track datasets, lineage and labels using Cloud Data Catalogue - which you can use to filter your resources - serverless pssst, its like collibra

# Executing Spark on Cloud Dataproc

Intro about Hadoop and HDFS  - distributed processing etc and MapReduce

* Spark is ~100z faster than MR due to in memory processing
    * declarative programming - You program what you want and it will do it the best way - vs- imperative programming 
* Problem with Hadoop is that the cluster needs to be tuned and optimised for the jobs which is time consuming and difficult 

Limitations:
* Not elastic
* Hard to scale fast
* Capacity limits
* No separation of storage and compute (you pay for everything and have to keep everything online - compute and hdfs is shared)

Cloud Dataproc: - simplifies the hadoop workloads on gcp
* Builtin support for Hadoop 
* Managed Hardware and config - no need to update and maintain, can add and remove nodes
* simplified version management
* flexible job config - can persist data off cluster so the cluster only is for processing

## Dataproc

A managed service for running Hadoop and Spark workloads:
* Cost effective - 1 cent per vCPU per cluster per hour  - can also use pre-emptible instance
* Fast and Scalable - start,scale and shut down only takes around 90 second each
* open source - Updates to hadoop and spark, and the hadoop ecosystem
* Fully managed
* versioning  - easy to change versions of hadoop, spark etc
* integrated - builtin with GCS, bq and bigtable and stack driver monitoring

Two ways to customise cluster: 
* Optional Components 
    * Selected via console or cmd line - e.g. Jupyter/Zeppelin notebook, Presto, Zookeeper
* Initialisation actions - Specifying executables that dataproc will run on all nodes in your cluster - immediately after setup

`gcloud dataproc clusters create <CLUSTER_NAME>  --initialization-actions gs://$MY_BUCKET/hbase/hbase.sh  --num-masters 3 --num-workers 2`

Dataproc has separation of compute and storage - compute clusters are only spun up when you need them for processing. You can store in a hdfs lke format in GCS buckets, or as tables in bigtable or bigquery

* Setup - console gcloud command on spin up from yaml file
* configure - region/zone, nodes type and numbers, preemptible etc (standard is one master, high availability is 3) initialisation scripts
* Optimise - Pre-emptible for lower costs, custom machine types and images, dataproc version etc
* monitor - stackdriver - all things can be monitored

## GCS instead of HDFS

HDFS has flaws: 64MB block sizes, no compute and storage separation, lots of data replication is expensive. GCP will have to transmit many blocks

* GCP has now Petabit bandwidth which allows for a bisection between storage and compute - you can process data where it is without transmitting
    * GCS, bq can store and Dataflow, Dataproc can compute
    * Very easy to use GCS instead of HDFS - just change your reference to gs:// instead of hdfs://
    * with this, clusters are now ephemoral

Performance best practices:
* Avoid small reads - use large block sizes where possible - see hdfs small file problem 
* Avoid iterating sequentially over many nested directories - since GCS is only simulating a file structure
* Use Distcp to get data to cloud

## Optimising Dataproc

* Location: Ensure region of data storage and cluster are as close as possible - dataproc has auto zone in the region you choose to place your cluster. But make sure it is close to your storage (bucket location)
* Network traffic - funneled through VPN and gateways? Could be bottlenecks
* Number of input files and hadoop partitions - adjust block size - fs.gs.blocksize
* Size of persistent disk
* Number of VMs to cluster
* Scheduled deletion of cluster after a given time period being idle - minimum 10 minutes and max 14 days to delete 
* Ephemeral clusters - pay for what you use 

Replicating persistent on prem setup is easy but has drawbacks:
* Expensive - not separation of compute and storage
* Difficult to manage with persistent clusters


### When to keep using HDFS
* Your job requires a lot of metadata operation - 1000s of partitions and dirs
* Modify HDFS data frequently or you rename dirs
* append operation is heavily used - or heavy IO and are especially sensitive to latency

### Different storage options
* GCS - Primary data store for GCP for unstructured data
* BigTable - sparse data, HBase compliant and low latency and high scalability
* Bigquery - Data warehousing


Cloud Dataproc has a workflow template that is a YAML file processed through a DAG

```sh
# the things we need pip-installed on the cluster
STARTUP_SCRIPT=gs://${BUCKET}/sparktobq/startup_script.sh
echo "pip install --upgrade --quiet google-compute-engine google-cloud-storage matplotlib" > 
/tmp/startup_script.shgsutil 
cp /tmp/startup_script.sh $STARTUP_SCRIPT
# create new cluster for job
gcloud dataproc workflow-templates set-managed-cluster $TEMPLATE \
    --master-machine-type $MACHINE_TYPE \    
    --worker-machine-type $MACHINE_TYPE \    
    --initialization-actions $STARTUP_SCRIPT \    
    --num-workers 2 \    
    --image-version 1.4 \    
    --cluster-name $CLUSTER
# steps in job
gcloud dataproc workflow-templates add-job \  
pyspark gs://$BUCKET/spark_analysis.py \ 
 --step-id create-report \  
 --workflow-template $TEMPLATE \  
 -- --bucket=$BUCKET
 # submit workflow template
 gcloud dataproc workflow-templates instantiate $TEMPLATE
```

Dataproc will then talk to YARN to autoscale accordingly to your job - but wont scale to zero - so not great for sparse utilisation or idle clusters, look to dataflow for that

You can set the driver log level using the following gcloud command:  
`gcloud dataproc jobs submit hadoop --driver-log-levels`  
You set the log level for the rest of the application from the Spark context. For example:  
`spark.sparkContext.setLogLevel("DEBUG")`  


# Manage Data pipelines with Data Fusion and Cloud Composer

Cloud Fusion:  Fully Managed Cloud native data integration service for quickly building data pipelines. Each instance is a managed environment and do not share any metadata or assets.
* Control Center - at a glance information
* Pipelines - Developer Studio
* Wrangler - Connections, Data Quality and Exploration - Data prep and quick transformations
* Metadata - Lineage and properties
* Hub - Plugins lists and pre-built pipelines

DAGs  - Directed Acyclic Graphs - One way pipelines - can be forked and non linear pipelines  
* The studio is where you can create these new pipelines
* Use preview mode to see how the pipeline will run - can correct errors in this mode in every node
    * Can monitor health of pipeline whilst running 
* You can also schedule batch pipelines and max number of concurrent runs
    * Note that data fusion is designed only for batch data pipelines
    * you can see the lineage of a given field value - and the lineage of operations applied to it 

## Wrangler - Exploring Data
Code free visual environment for transforming data in data pipelines - and exploring datasets visually for insights
* Transformation steps are called - Directives - and they stick together pipeline stages

![wrang](./batch_pipelines/images/wrangler.PNG =700x)

You can add new directives to transform the data - once happy with your transformations you can then create a pipeline to schedule to run at intervals 


## Orchestrating work between GCP services - Cloud Composer
Basically just Airflow!  Open Source

Orchestrates automatic workflows - It commands the GCP services that we need to run  

* It is a serverless environment for running managed Apache Airflow Environment instances - with each env as a separate webserver and folder in GCS for the DAGS
* The UI webserver and GCS DAGS folder is accessible through the UI. GCS DAGs folder is where the code will actually be stored. 

Airflow workflows are written in Python - with one .py file for each DAG
* Airflow uses operators in DAg to orchestrate GCP services - one operator per task.
    * BigQuery operators are popular for updating your ML training dataset
        * create empty table, delete dataset, to cloud storage operators etc
        * Can specify SQL query to run against bq which can be parametrised in python
    * Cloud AI Platform (Formerly Cloud ML Engine) Operators launch training and deployment jobs
        * ML Batch Prediction, training, model operators - Note that even though the name changed the operators are still MLEngineBatchPredictionOperator
    * AppEngine Operatorsto deploy and redeploy models

Airflow dependencies - t2.set_upstream(t1)  etc t2 wont run until t1 has completed

## Scheduling for Cloud Composer Workflows
* Periodic - Pull
    * In code you can say 
        with models.DAG(schedule_interval=datetime.timedelta(days=1),)
* Event-Driven - Push
    * Driver is Cloud Function that we can create - e.g watch GCS bucket for new CSV Files, based off a trigger (HTTPS request, PubSub, FireStore)
        * Good for Stock data, disaster data etc
    * Cloud Function function name to execute is type sensitive! 

![wrang](./batch_pipelines/images/cloud_func.PNG =700x)

You can monitor health by viewing logs in the airflow UI and run logs. But also in Stackdriver to see logs as well. 

# Cloud Dataflow - Serverless Data Processing 
