# GCP Big Data and Machine Learning Fundamentals

There are four fundamental aspects of GCP core infrastructure and a top layer of products and services you interact with:
* Top Layer: Big Data and ML Products
* Compute Power, Storage, Networking
* Security

Many ML Products leverage the trained AI models from the vast quantity of data that google houses.  - saves you training time 
* Sight: Cloud Vision, Cloud Video Intelligence, AutoML Vision
* Language: Cloud Translation, Cloud Natural Language, AutoML Translation, AutoML Natural Language
* Conversation: DialogFlow, Cloud Text-to-speech, speech to text etc

# Storage
> `Storage: gsutil mb -p [PROJECT_NAME] -c [STORAGE_CLASS] -l [BUCKET_LOCATION] gs://[BUCKET_NAME]/`  
This will make a bucket 

There a four storage classes:
* Standard Storage - hot data
* Nearline - once a month or less
* Coldline - low cost once a quarter
* Archive - dr

Read some of the storage class md files from the associate as well

# Resource Hierarchy
Bucket ids and gcp projects have to be globally unique 

* Organisation - root node of the gcp hierarchy
    * Folders - can have collections of projects
        * Projects - logically organise resources
            * Resources (bigquery, cloud storage buckets, compute engine instances)

Use IAM - Identity Access Management can fine tune the access to the gcp resources 

The good thing about GCP is that it handles the lower level securities like networking, hardware security etc
e.g 

With BigQuery - data is encrypted with a data encryption key, these keys are then encrypted again with key encryption keys - known as ENVELOPE ENCRYPTION

# Compute
* COmpute engine for VMs
* GKE Kubernetes for clusters of machines running containers
    * K8s orchestrates code that runs in containers
* App Engine - fully managed PaaS platform as a service framework
    * Typically used for long lived web apps that can autoscale from millions to billions of users 
* Cloud Functions - serverless execution env - function as a service
    * Pay for what you use only 
    * Used for code triggered by an event - e.g. new file hitting gcs
    
# Use case - migrating an on prem application to GCP
* Utilising and tuning on prem clusters
* Off cluster storage with gcs

* cluster users and items to combat sparsity

Choose your storage solutions based on access pattern 

|                    | Cloud Storage | Cloud SQL | Datastore | BigTable | BigQuery |
|--------------------|---------------|-----------|-----------|----------|----------|
| Capacity           | Petabytes     | Gigabytes | Terabytes | Petabytes|Petabytes |
| Access Metaphor    | Like files in file system   | RDBMS     | Persistent Hashmap NoSQL| Key-value, HBase | Data Warehouse | 
| Read               | Copy to local disk | SQL | filter objects | scan row | SQL | 
| Write              | One file | INSERT | put object | put row | Batch/stream
| Update Granularity | an object | Field | Attribute | Row | Field
| Usage              | Store blobs | No-ops SQL db on cloud | Structured data from AppEngine | No-ops append only, no txn | Interactive SQL fully managed data warehouse

![External and Internal Load Balancers](./images/dataoptions.PNG =600x)

## CLoud SQL

* Familiar
* Flexible pricing
* Managed backups
* Auto replication
* Fast connection  -same region
* Secure

DATAPROC = managed service for creating Hadoop and SPark clusters and managing data processing workloads
dataproc - has autoscaling to provide the flexible capability - based on hadoop yearn metrics - these can also be pre-emptible machines for lower cost (80% cheaper)
* this way with storing data outside the clusters, we have flexibility on the resources without losing data


## Separation of compute and storage

Enables cost effective scale for workloads

Off cluster storage - the speed of googles data center enables this without losing latency. 

Google has petabit bisectional bandwidth (rate of comms at which servers communicate with other servers futher away) - this means that locally copying data around isnt worth it and the data can be transformed where it is stored!

Changin your code from hdfs to gcp is easy  -just point from hdfs:// to gs://
* There are different connectors for different data stores as well, HBase for bigtable, BigQuery etc

## Summary Dataproc
* Hadoop without cluster management - fast cluster within 90s
* Lift and shift existing Hadoop workloads
* Connect with cloud storage to separate compute and storage 
* Re-size clusters effortlessly, Pre-emptible VMs for cost savings

# BigQuery
Big query is actually two big services in one:
* A fast SQL Query Engine
* a managed storage for datasets

Bigquery is a petabyte scale fully managed data warehouse  
It is designed to be a good staging area for processed data from real time events, app engines and dataflow - to serve to visual apps like tableau, excel etc
Big query UI has something called Slot time consumed: unit in time - this is the none parallelised time for a query. say 50 minutes of slot time can have only 10 seconds of elapsed time
* serverless
* flexible pricing model - pay as you go
    * cached results as well which is free
    * There is also a subsription model as well
* data encryption and security
* geospatial data types and functions
* foundation for BI and AI

## bq hierarchy
Data is contained within a project in datasets in tables. A dataset is synonymous to the classic database object within say mysql, There are stored as highly compressed columns in the colossus file system. bq also supports both bulk data ingest and streaming. It is very compatible with other services using connectors

![External and Internal Load Balancers](./images/bq_hierarchy.PNG =600x)

bq gives you 1 tb of compute and 10gb storage for free every month

## syntax
* backticks `` are bq's way to indicate table object, good for when you need to esacpe characters, e.g. hyphens
* You can highlight and click a table whilst holding the window key to view a table schema (free if the data is cached / inherited)
    * you can even preview data without doing select * limit 10

## Cloud Dataprep: Cleaning and transforming data with a UI
You can use a UI to inspect dataset quality using Cloud Dataprep - this is a GCP product offered with Trifecta. 
* Dataprep can load a sample of bq dataset into exploration view 
* This results in some column histograms - null calues, counts etc - its like alteryx a bit
* You can create a pipeline in dataprep and run it through cloud dataflow to transform your data 
* The main advantage is just that you have a fancy UI with builtin scheduling

* You can schedule queries by using @run_time parameter or the query scheduler in BigQuery UI 

## IAM in bigquery
These are the primitive roles - note that default access to datasets can be overriden on a per dataset basis
* Viewer: can start a job, list jobs 
* Editor: Create any new dataset in project
* Owners: can list all datasets, and delete etc  
Remember to give the min permissioned needed for the role - and to use separate projects for different envs (dev, prod, preprod etc) and to audit the roles

## BigQuery as a managed storage for datasets 
the native bq storage ensures highest performance - this is auto scaled and fully managed, backed up.
* aside from native storage - you can query data sources in gcs and drive - e.g. raw csv in a bucket or sheet - you can write a query.
    * common use case is say having an external data source that is small and constantly chaning like a price list 
        * However consistency is not guaranteed

You can also stream to ingest using the API. There are restrictions: the max row size is 1 MB adn teh max throughput is 100,000 records per second per project. Any more you need to use say cloud big table or data flow if you need the data to be transformed or aggregated midflight

## Structs and arrays
* A good thing about bq is that it supports arrays as a single row,  and you can split those arrays up within that row. This way you can have a one to many row association. This is good to avoid complex joins. It also supports STRUCTS which you can think of as a family of arrays (or column families) - almost prejoined

# Normalisation vs denormalisation for reporting 
* Normalised datasets break up a huge parent table into compoennt child tables, you're storing one fact in one place and not repeating yourself across records. To bring all the data together you just write one big join and specify all the tables to bring in. This is common practive for relational databases like mysql or sql server. 
* denormalised is the opposite is used to combine the multiple table data into one so that it can be queried quickly 

# Choosing a ML model

![External and Internal Load Balancers](./images/ml_model_choose.PNG =800x)

## ML in SQL
You can do ML on your structured datasets in bq using SQL in minutes
* create a model with a sql statement (create model numbikes.model options(model_type=`linear_reg`, labels=[`num_trips`] as with bike data as (select )))etc 
    * CAN ALSO DO CREAT OR REPLACE MODEL
    * can inspect ml.weights for each feature
    * you can use evaluate
* write a sql prediction query and invoke ml.Predict

bq will adjust the learning rate and data splits for you automatically with default values - but you can customise them if you want
compatible with both linear regression and logistic classification


![External and Internal Load Balancers](./images/bq_ml_cheatsheet.PNG =800x)


# Real time Dashboards with Pub/Sub, Dataflow and Data studio

## Pub/Sub Publisher Subscriber Model
Basically publishing messages to its subscribers - it is a distributed messaging service from upstream iot (gaming, streams, sensors)
* It is scalable to demand and the service offers e2e encryption

![External and Internal Load Balancers](./images/pubsubeg.PNG =800x)

Above is an example of a typical pipeline - upstream data is ingested into pubsub - it reads stores and publishes out to subscribers of the data topic the messages
* Cloud dataflow is a subscriber of pubsub, it then ingests and transforms those messages in an elastic streaming pipeline and output the results into an analytics data warehouse like bq. 

### Pubsub topics

A ps topic is like a radio antenna - publishers can send data to a topic that has no subscribers (receivers). Like a radio broadcasting at a frequency. Pubsub is the radio, the topic is the frequency and the listeners are the subscribers

Per topic, there are not limits to the numbers of pubs and subs

it is great for many upstream sources and manymore downstream subscribers

## Apache Beam
 
Unified and portable data processing programming model for both batch and stream - written in `Java, Python or Go`
* Unified - single programming model for both batch and streaming use cases
* Portable - pipeline executed on multiple envs
* Extensible - SDKs for writing  custom data pipelines - io connectors and libs are open source
* runner to run distributed processing
* open source

Beam will create a model representation of your code which is portable across runners (like compiling?) The runners pass off the model for execution on different engines e.g. Dataflow will run apache beam as an engine (but spark and flink also work)

# DE Consideration when building pipelines


| Pipeline Design                        | Implementation               | 
|----------------------------------------|------------------------------|
| Code work with batch and stream?       | Maintenance and Overhead?    |
| SDK support the transformation I need? | Infrastructure reliability?  |
| Existing solutions?                    | Scaling handling?            |
|                                        | Monitoring and Alerts?       |
|                                        | Locked into a vendor?        |

With dataflow a lot of this implementation is handled for you, little maintenance, google infra, autoscaled, integrated with stackdriver   
Serverless and noops

So what does Cloud Dataflow do with a job?Once it receives it, the Cloud Dataflow service:
* Optimizes the execution graph of your model to remove inefficiencies 
* It schedules out work in a distributed fashion to new workers and scales as needed
* It will auto-heal in the event of faults with workers
* It will re-balance automatically to best utilize it’s workers
* And it will connect to a variety of data sinks to product a result. BigQuery being one of many sinks that you can output data to as you will practice in your lab. 

![External and Internal Load Balancers](./images/dataflow_sum.PNG =400x)

Within bq itself there is data studio - where you can immediately start exploring your data visually after you execute a query
* Note that access to dashboard doesnt mean they have access to the data

# Unstructured data using ML

Rule of thumb - only consider custom model building with around 100,000 + to millions of data points. If not then better to use a prebuilt model - e.g. DialogFlow for chatbots or AutoML for more specific blocks of work
* These pre-built apis are offered as services such as : NLP, Speech-to-text, Vision, DialogFlow, Translate. These are heavily embedded into other software, e.g. google sheets can invoke the translate api
* example of DialogFlow - more ML based conversation for chatbots - it is an NLU - Natural Language Understanding engine - to process and understand natural language input. With builtin enetity recognition, it can identify and label by types such as person, location etc, and process logic for them
    * It can be trained continuously with more and more data the more experience it has
    * Dialog enables you to 
        * Build faster
            * pre-built agents and examples - like food delivery,  hotel booking, Maps etc
        * Engage efficiently
            * builtin nlu
        * maximise reach
            * connectable with all sorts of agents - fb messenger, twitter etc
            * deploy anywhere
            * 20+ languages supported 

## AutoML
Custom ML Models - Pre-built with AutoML

* Say you want to identify the types of clouds in the sky - with vision api - it might infer that it is seeing sky/cloud/blue however it wasnt trained to recognise types of clouds of this level of granularity
* AutoML can extend the capability of the AI building blocks - we can use out own labelled dataset of cloud images and train a custom model using `AutoML Vision` - which is codeless and simple and intuitive to use as a gui

Precision - number of photos correctly classified as a particular label / total photosof a label
Recall - number of photos  classified as a particular label / total number of photos of that label
There is atradeoff between these two attributes

It also gives you a confusion matrix as well

![confu](./images/conf_mat.PNG =400x)

Auto ML will actually train and evaluate many models at once and compares them - iterating over 20k times to find the most accurate NASNet (Neural Architecture Search)

![confu](./images/automlvsvision.PNG =700x)

Note you can create custom data science models - using notebooks, GCP has ML Engine which is like ipy notebooks
* Within the notebook you can even diorectly run bq sql statements with `%%bigquery --project %PROJECT` and then your sql