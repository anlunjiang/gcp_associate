# Building Resilient Streaming Analytics Systems on GCP

# Processing Streaming Data 
* Streaming allows us to make decision in realtime - or for use cases where data loses value with time 
    * e.g. Fraud detection, Gaming events 
    * Streaming is data processing for unbounded data sets (infinite data set and never complete- time is important)

Example Architecture:
* Cloud Pubsub --> DataFlow running Beam --> Bigquery/BigTable


            Cloud Composer will orchestrate the following:

>     Enduser apps/Databases  --> Ingest and distribute with Pub/Sub --> DataFlow Streaming --> Write to Data Warehouse --> Dataflow batch 
>        produce events           Asynchronous messagin bus              Aggregate, enrich        / Run through ML Model     Backfill or reprocess
>                                Can hold events until consumed             data
>                                by services for processing


## Cloud PubSub - Serverless Messaging 

Fully managed data ingestion and distribution system - nothing to install as well :
* Availability
    * Run in most servers in many areas of the world - offering fast global data access
* Durability
    * Will save your messages for up to 7 days if your systems are down and not able to process them in time 
        * Can retrieve messages at a later tume and continue with processing 
* Scalability
    * serverless and provides infra to provide scalability
    * End to end encryption in transit and at rest
* Builtin buffer for especially high loads and traffic spikes
    * However this is wasteful


The sender and receiver do not both have to be online at the same time - e.g. email - the message still goes through and the receiver will receive when its online - if subsribed to the topic.

Pub Sub is split into two structures:
* Topics

* Subscription

* Client that creates the topical is called the publisher and the receiver is a subscriber of a topic
* Publishers will publish events and messages into a topic 
* To receive messages - you must create a subscription to that topic
* Topics can have many subscriptions - but a given subscription belongs to a single topic 
* services are subscribed to that subscription that is associated with a topic
* Hierarchy goes: Publisher - Topic - Subscription - Subscribers

## Push and Pull

* Pull Delivery: Subscribers are periodically calling for messages - and pubsub delivers the messages since the last call 
    * Subscriber sends a pull request to the subscription - which sends the messages - subscriber will then acknowledge that message 
    * Messages are stored for up to 7 days
* Push Delivery - Pub/Sub sends each message as a HTTPs request to the subscriber application at a configured endpoint
    * The push acknowledgement is just a 200 OK 

![pushpull](./streaming_analytics_systems/images/pushpull.PNG =600x)

* Acknowledgements ensure that every message gets delivered at least once on a per subscription basis
* Messages are resent if the subscriber takes more than `ackDeadline` to respond
    * This deadline can be extended by the subscriber 
* Subscriptions have one endpoitn to receive acks, but they may send to a web app that autoscales
    * Only one client needs to ack the receipt in this case

## Publishing with Cloud Pub/Sub

* `gcloud pubsub topics create sandiego` creates the topic  
* `gcloud pubsub topics publish sandiego "hello"` Publish to a topic
 
```python
import os 
from google.cloud import pubsub_v1

publisher = pubsub_v1.PublisherClient()
topic_name = 'projects/{project_id}/topics/{topic}'.format(    
    project_id=os.getenv('GOOGLE_CLOUD_PROJECT'),   
    topic='MY_TOPIC_NAME',
)
publisher.create_topic(topic_name)
publisher.publish(topic_name, b'My first message!', author='dylan')
# pubsub only sends raw bytes - not restricted to just text - images can be serialised
# you can send attributes like author in the message - so downstream can receive metadata
```
Limit is 10MB

## Subscribing with Pub/Sub using Async Pull

Async pull provide higher throughput in your app - it doesnt ever block messages - good when low latency is a priority

```python
import os
from google.cloud import pubsub_v1

subscriber = pubsub_v1.SubscriberClient()
topic_name = 'projects/{project_id}/topics/{topic}'.format(
    project_id=os.getenv('GOOGLE_CLOUD_PROJECT'),
    topic='MY_TOPIC_NAME',
)
subscription_name = 'projects/{project_id}/subscriptions/{sub}'.format(
    project_id=os.getenv('GOOGLE_CLOUD_PROJECT'),
    sub='MY_SUBSCRIPTION_NAME',
)
subscriber.create_subscription(
    name=subscription_name, topic=topic_name
)
def callback(message):
    """
    Callback when message is received
    """
    print(message.data)
    message.ack()

future = subscriber.subscribe(subscription_name, callback)
```

## Synchronous Pull with Cloud PubSub

Can pull messages from the command line:
* `gcloud pubsub subscriptions create --topic sandiego mySub1`
* `gcloud pubsub subcriptions pull --auto-ack mySub1`

```python
import time
from google.cloud import pubsub_v1

subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path(project_id, subscription_name)
# `projects/{project_id}/subscriptions/{subscription_name}` subscription_path format
NUM_MESSAGES = 2
ACK_DEADLINE = 30
SLEEP_TIME = 10
# The subscriber pulls a specific number of messages.
response = subscriber.pull(subscription_path, max_messages=NUM_MESSAGES)
```

* By default - publisher batches messages - this can be turned off if you desire lower latency, but lowers throughput
* Batch can send 10-50 messages at a time and increase efficiency
* The batch setting can be changes in python
    * client = pubsub.PublisherClient(batch_settings=BatchSettings(max_messages=500))
* Note that in pubsub message order is not guaranteed, as with latency and duplication
    * This is a compromise made for scalability 
    * earlier messages may arrive later due to routing
        * Hence not suitable for say chat applications

Cloud Pub/Sub with Dataflow - Exactly once, ordered processing
* PS delivers at least once
* Dataflow - Deduplicates based on message id (dupes in ps will have the same id), orders (handles late data but not true order by sent ts) and windows the data

# Cloud Dataflow Streaming Features

Challenges with processing streaming data
* Scalability
* Fault tolerance - despite large volumes
* Model - is it streaming or repeate batch
* Timing - late data handling

Aggregating unbounded datasets:
* The stream is divided into a series of finite windows 
    * The window is defined by the element data-timestamp (usually PS ingest time)
    * This can be done automatically via dataflow 
    * Can modify it so it aggregates on a custom timetamp as well
        * `yield beam.window.TimestampedValue(element, unix_timestamp)`

## Dataflow Windowing
3 Kinds of windows fit most circumstances:
* Fixed - divides data into time-based finite chunks, e.g. day, month - consistent non overlapping intervals
* Sliding - required when doing aggs over unbounded data - can overlap - e.g. running average 
* Sessions - defined by minimum gap duration - timing triggered by another element - good for communication in bursts , web sessions from a user
    * time of 10 minute or more is a session for example

![pushpull](./streaming_analytics_systems/images/dataflowwindows.PNG =800x)

Handling latency and windowing:
* System will calculate lag by taking expected arrival time difference against actual arrival time 
* Dataflow keeps track on lag with Watermarking
    * Provides flexibility for a little lag time - the watermark time is the limit cutoff for a window 
* Late data can be discarded by default, or you can reprocess the window based off the late arrivals
    * only java supports late data - not python (in the works)

![pushpull](./streaming_analytics_systems/images/watermark.PNG =600x)

Triggers for aggregate functions:
* AfterWatermark: Default behaviour, trigger functions at the watermark, but custom triggers are allowed 
* AfterProcessingTime: Processing time triggers aggs, say every 30 seconds, and the window never ends but interim results are produced
* AfterCount: Data driven triggers  - every say 15 elements the data will be processed 

![pushpull](./streaming_analytics_systems/images/triggers.PNG =600x)

Accumulation modes:
* Accumulation - show new processed values and old ones in an array
* Discard - discard old values and only present newly processed values

# BigQuery and BigTable Streaming Features

* bq allows you to stream records into a table: the query results can incorporate the latest data - 100K rows per second for inserts 
* Data studio within bq can visualise insights - which can connect to different data sources - sheets, bq, mysql
* remember just because someone has access to your dashboard doesnt mean they can see the data 

## BigTable

* NoSQL queries over large petabyte datasets in milliseconds and microseconds for specific cases
* Very low latency alternative to bq and higher throughput
* Good for non structured key value data, time series, non transactional, rapidly changing
    * Good for real time lookup as part of an application - where speed and efficiency are desired 

Cloud BigTable stores data in a file system called `Colossus`
* Colossus contains tablets that are data structures used to manage data. Tablet metadata is stored on the vms in the BT cluster
    * This way repointing nodes to different tablets is fast - since only metadata is changed 
* It learns = can detect hotspotting where a tablet has many activities:
    * It can split the table into 2. 
    * Re-balance the processing by moving a tablet pointer to a different VM
* BigTable Clusters store metadata of tablets - A VM will process its tablet which contains data
* Fault Tolerant - in case of failure - if a Cluster node fails - only the pointers to the data is lost - The actual data is still safe in Colossus
    * A new node will be spun up with a fresh copy of the metadata - which was handled by the faulty node previously 


### Bigtable design

Design principle of speed through simplification - NoSQl Database

* Stores data in columns - with the Row-Key as the index - the ONLY one index in the table (no secondary indexes)
* Speed depends on both your data and row key
    * Scan - very fast since it looks through the row key - consistent speed
    * Sort - Variable to: The order of the data originally, data to sort
    * Search - bounce between row key and column data
    * Speed up by sorting data before loading to BT, and select a row key that minimises sorting
        * Custom rowkey to tailor to your queries - can make a column that is from two indexes
* Column Families - easy to group columns and pull only relevant information out 

![pushpull](./streaming_analytics_systems/images/colfam.PNG =800x)

* query with a relevant custom row key with the data presorted (e.g. reverse timestamp to show the latest data if that is more common)

## Changing data in BigTable
Say you want to update a row:
* A row is marked for deletion - it is skipped for any subsequent processing - but not immediately removed 
* New row is appended sequentially to the end of the table - both rwos actually exist for a period of time 
* Compaction then occurs periodically - which removes rows marked for deletion and reorganises the data for Read/Write efficiency

Optimisation of data by organisation
* Group related data for efficient reads - column families - up to 100 without losing performance 
* Distribute data evenly for more efficient writes - avoid hotspots for row_key index
* Place identical values in the same row or adjoining rows for more efficient compression
* BT self learns and improves by learning access patterns - periodic reorganisation ensures maximum efficiency and speed
    * remember rebalancing is not time consuming as you are just repointing using metadata, test with data and give time
* Clients and bt should be in the same zone
* Disk speed on VMs in cluster - ssh instead of hdd
* Increase throughput by node count - linear scale
* smaller rows can increase throughput as well - but data dependant
* You can use Key Visualizer which is a tool that can expose read write access patterns  - to find hotspots, if rows have too much data - or if your index is balanced

Replications improve BT availability:
* Provide near real time backups - different clusters for different types of requests
* Global Presence
* Increase availability
* Isolate servings apps from batch reads

# Advanced BigQuery Functionality

Best Practices:
* Get used to using Structs as they are a good way to arrange your columns - almost like col families - 
```SQL
SELECT STRUCT(
    start_stn.name,
    start_stn.longitude,
    start_stn.latitude,
) as starting,
STRUCT(
    end_stn.name
) as ending,
FROM TABLE
```
* use with select cols as table -  good for where conditions since the where will be included in the with clause when filtering - called `automatic predicate pushdown`
* Do not SELECT *, filter early and often
* APPROX_COUNT_DISTINCT is faster than count(distinct) - this is a faster built-in function
* Partition tables - remember 3 ways (ingestion time, col of type timestamp or data, integer based col)
    * Cuts down data being queried
* Clustering - sorts data bsed on values in the clustering columns
    * Good to setup clustering at table creation time
    * Good when you use lots of aggregations against columns or filters
    * Already partitioned good to cluster already
* bq will recluster over time especially with streaming since new data is appended
* Execution details can allow you to track IO counts 
* Analyse BQ performance in stackdriver 

![pushpull](./streaming_analytics_systems/images/optimise_bq.PNG =600x)


## GIS Functions (Geographic Information Systems)

* bq supports GIS functions (like mssql) like `ST_DISTANCE()` to get the straight line distance between two geographic points (long/lat)
    * ST functions are Spatial type functions are are common in geographical use cases
* `ST_GEOGPOINT()` take lat and long float values and converts it to an actual GIS point 
    * `ST_DISTANCE(ST_GEOGPOINT(lat, long), ST_GEOGPOINT(lat2, long2)) as meters`
* `ST_MAKELINE()` draws a line between two points in a GIS vis tool
* `ST_MAKEPOLYGON(ST_MAKELINE(point1, point2, point3))` make a shape from ST_LINES
* `ST_INTERSECTS` Do two lines/shapes intersect
* `ST_DWITHIN` checks if two locations objects are within an arbitrary distance apart
* GEOVIS is a web based frontend that you can run bq queries on to visualise geography results - you just got to bigquerygeoviz.appspot.com and paste your project ID in

## Analytic window functions for advanced analysis
* Standard Aggregations
    * e.g. count, sum, avg, min, max etc
* Navigation functions
    * lead (value of row n rows ahead of current), lag, nth_value, first_value
* Ranking and numbering functions
    * RANK() OVER(PARTITION BY col ORDER BY col DESC) as rank where rank == 1 
        * RANK function aggregates over groups of rows
    * CUME_DIST, DENSE_RANK (ties do not affect rank)

## WITH Clause and Subqueries
* WITH is a simply named subquery (CTE common table expression in MSSQL)
* It basically is a temp table good for breaking complex queries
* Multiple subqueries can be chained together with a single with - future subqueries can reference existing ones

# BigQuery Pricing

You are charged as soon as you load data into BQ, there are two types of pricing
* Active Storage Pricing
    * Pro-rated per MB/second
* Long-term Storage Pricing
    * table/partition not edited 90+ consecutive days
    * pricing drops ~50%

Pricing models include
* On-demand pricing
* flat rate pricing

Guideline is 2000BQ slots for every 50 complex queries