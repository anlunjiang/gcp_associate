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

A Serverless execution service for data processing pipelines written in Apache Beam - which is very portable and not locked to data flow 

Dataflow can have the same code to do both batch and streaming data pipelines (Batch + Stream = Beam)


| Use              | Cloud Dataflow                                              | Cloud Dataproc
|------------------|-------------------------------------------------------------|---------
| Recommended for: | New data processing pipelines, unified batcha and stream B1 | Existing Hadoop/Spark Apps
| Fully Managed    | Yes                                                         | Yes
| Auto-Scaling     | Yes, transform-by-transform (adaptive)                      | Yes
| Expertise        | Apache-Beam                                                 | Hadoop Ecosystem

* Dataflow is serverless - so you dont have to manage clusters at all - everything is managed for you 
    * Scalable and low latency
* With a new data processing pipeline - consider using Data Flow first 
* Existing Hadoop pipeline - migrate to dataproc first and then over time to data flow 

![wrang](./batch_pipelines/images/flowvsproc.PNG =700x)

Apache beam unifies programming and processing - with 4 mains concepts:
* Pipelines
    * Identifies data to be processed and actions to be taken
* PCollections
    * Data is held in a distributed data abstraction called Pcollections
    * Immutable - any changes creates a new PCollection - the old one is unchanged 
* PTransforms
    * Actions or code are contained in an abstraction called PTransforms
    * Handles IO and transformations
    * Data passes along PCollections and PTransforms along the pipeline
* Pipeline Runner
    * Container hosts - like K8s engine

![wrang](./batch_pipelines/images/beam.PNG =600x)

A PCollection (Parallel Collection) represents batch or stream data with no size limit and distributed across many workers - the more data the more its distributed, there are two types:
* Bounded PCollection - data has a fixed size  - not, that the PCollection size is limited
* Unbounded - data is not fixed in size
* PCollections dont have to be read from external source - you can use python to write to a PCollection from memory: beam.Create(var name)

* All data types are stored as serialised byte strings - no need to serialise of de serialise before - this is done automatically / - Elements in PCollections are immutable
* Data flow rebalances work and optimises constantly - can fuse transformations together - breaking the jobs into units of work and schedules to workers
    * **Dynamic rebalancing** - if nodes are taking a long time and others have finished - the remaining work is auto rescaled - savings time and money 
    * Autohealing and monitoring using UI
* Intelligent watermarking - Dataflow can handle late and lagged data 

![wrang](./batch_pipelines/images/dataflow.PNG =600x)

### Constructing a Dataflow Pipeline

* In Python: `PCollection_out = (PCollection_in | PTransform_1 | PTransform_2)` 
    * Pipe symbol is used - with T1 before T2 - note that PCollections are made between each PTransformation - but we only reference the last one  - not Inplace at all
* In Java we use .apply(PTransform_1).apply(PTransform_2) instead of pipe
* Branching is possible by just sending the same PCollection through two different PTransforms 

![wrang](./batch_pipelines/images/branching_pcollections.PNG =600x)

```python
import apache_beam as beam
if __name__ == '__main__':
    with beam.Pipeline(argv=sys.argv) as p: # Create a pipeline parameterized by command line flags
        (
            p | 
            beam.io.ReadFromText('gs://...') | # Read input
            beam.FlatMap(lambda line: count_words(line)) | # Apply transform
            beam.io.WriteToText('gs://...') # Write output
                )  
     # end of with-clause: runs, stops the pipeline 
```

* beam io has many connectors to read from Cloud GCS, pub/sub, bigquery beam.io.ReadFromText("bucketname")
* It can also write to bq

### Map and FlatMap
* Map for 1:1 relationship between IO  
`'WordLengths' >> beam.Map( lambda word: (word, len(word)) )`  
* FlatMap for non 1:1 relationships, usually with a generator
    * Similar to Map but fn returns an iterable of zero or more elements - the iterables are flatterned all into a single PCollection
 ```python
 def my_grep(line, term):   
        if term in line:      
            yield line
# 1:many relationship 
'Grep' >> beam.FlatMap( lambda line: my_grep(line, searchTerm)
```

### ParDo

ParDo implements parallel processing within an itermediate step - it acts on one item at a time in the PCollection  
* Multiple instances of class on many machines and does not contain state
* Used for filtering datasets, formatting, computing etc

ParDo requires code passed as a DoFn Object:  class that defines a distributed processing function:

* DoFn must be idempotent, threadsafe and serialisable

```python
# Applies a ParDo to the PCollection words, to compute lengths for each word

words = ... # input is a PCollection of strings

class ComputeWordLengthFn(beam.DoFn): # DoFn performs on each element in the input PC
    def process(self, element):
        return [len(element)]

word_lengths = words | beam.ParDo(ComputeWordLengthFn()) # Output is a PCollection of Ints
```

## After Map phase - Group By Key

GroupByKey - explicitly shuffles key-value pairs - to group together like keys
* Works on a PCollection of Key-Value Pairs returning a single keyvalue pair where values are grouped

```python
cityAndZipcodes = p | beam.Map(lambda fields : (fields[0], fields[1])) # Frrst ParDo to get the Keyvalue pairs
grouped = cityAndZipCodes | beam.GroupByKey() # group by keys 
```

* However Data skew makes grouping less efficient at scale - if your group by key is skewed then all the work will go to a hotspot of workers and the other workers will be idle:
    * Thus try to avoid grouping until your data is all transformed and nonskewed

![wrang](./batch_pipelines/images/skew.PNG =600x)

* CoGroupByKey is similar - it joins two or more key-value pairs with the same key (Two PCollections with the same key to group and result contains both their values)

![wrang](./batch_pipelines/images/cogroupby.PNG =600x)

## After GroupBy - you need to Combine (Reduce) a PCollection

This is applied to a PCollection of values and aggregates them by a agg function (sum, count etc) - can apply to keyvalue pair by CombinePerKey(sum)

![wrang](./batch_pipelines/images/combine.PNG =600x)

You can create custom CombineFn as a subclass and override existing operations

```python
class AverageFn(beam.CombineFn):
```

You must method override four methods to create your own custom CombineFn:
* create_accumulator
* add_input
* merge_accumulators
* extract_output

Generally Combine is more efficient than GroupByKey - since DataFlow knows how to parallelise a combine step
* GroupByKey uses only one worker per key and requires shuffling which is expensive
* Combine allows for distribution of keys across multiple workers and process in parallel and is faster

## Flatten and Partition

* Flatten - Works like a SQL Union - it unions many Pcollections that store the SAME DATATYPE into one large Pcollection 
* Partition - Splits PCollections into smaller PCollections. e.g. split by quartile and 1 quartile of data requires different processing that others 

Side inputs inject additional runtime data - DoFuntcion can access and process. You can create a view of some other data that fits in memoery 

## Every PCollection is processed within a window 
* The default window is called the global window - starts when data is input and ends when the last element in collection is processed.
* In bounded PCollections - the elements are all marked as occuring at the same time and we know when the last one is finished
 * In unbounded datasets - there is no predetermined end - so the global window is not useful
    * Global window is not useful for say - streaming data
    * ParDos like Combine and GroupByKey would never perform the shuffle step at the end

* Instead of global window you can use Timebased windows:
    * each element has a timetamp and the elements are grouped by the timestamp - each group would get its own average timestamp
    * There are different types of timebased windows such as sliding windows - start anew window every say 60 seconds

## Dataflow templates

Enables rapid deployment of standard job types - removing need to develop pipeline code 
* Template workflow supports non developer users 
* Can execute templates from console, command line and rest API

## Dataflow SQL
* Lets you use SQL Queries to develop and run Dataflow jobs from the BigQuery UI.
    * Can associate schemas with objects like tables, files and pubsub topics
    * This is in Alpha though currently 
