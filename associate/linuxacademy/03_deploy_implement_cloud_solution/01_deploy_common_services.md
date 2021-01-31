# Application Deployment

[[TOC]]

# Configuring Common Services

This section will follow on the linux academy codebase found in this repo as `content-google-cloud-engineer-master` (cgcem)

## Enable APIs

[enableapis.sh](../../content-google-cloud-engineer-master/common/build/enableapis.sh)

> `gcloud services enable <api_name_id>` e.g. storage-component.googleapis.com

## Network setup

[network.sh](../../content-google-cloud-engineer-master/common/build/network.sh)

Creates a VPC with two subnets, one for the products and one for the ads app

> `gcloud compute networks create <network_name> --project=<proj_name> --subnet-mode=custom`

```bash
gcloud beta compute networks subnets create <subnet_name>  
--project=$PROJECT_NAME   
--network=$SERVICES_NETWORK \  
--region=us-central1 \
--range=10.29.0.0/24 \ # CIDR Notation Classless Interdomain Routing
--enable-private-ip-google-access \
--enable-flow-logs
```
* flow logs will enable us to log the flow of traffic in the network
    * Great for debugging and analysis
    * Can be expensive to store since there are a lot of logs
* This creates a subnet with a custom defined ip range
* It then creates firewall rules to allow internal traffic on network to flow

> `gcloud compute networks (subnets) list` View all created (sub)networks in the project

## Creating Cloud Storage Bucket

We dont use the gcloud component for cloud storage: we use `gsutil` command

[privatebucket.sh](../../content-google-cloud-engineer-master/common/build/privatebucket.sh)

> `gsutil mb -p $PROJECT_NAME -c regional -l $PROJECT_REGION gs://$PRIVATE_ASSETS/` gsutil make bucket  
mb: make bucket,  -c: storage class, -l: which region gs://: name of bucket

* Google creates all buckets as `private` as default unless explicitly stated
* gs:// makes sure gsutil knows this is a cloud storage resource, not a local file

> `gsutil ls` list all buckets - similar syntax to bash syntax

[publicbucket.sh](../../content-google-cloud-engineer-master/common/build/publicbucket.sh)
Used to serve images inside the frontend application
* Note that the URL of our frontend application will not match the URL of cloud storage (different domain)
    * The headers cloud storage sends the browser need to be adjusted so that the browser knows the application is allowed to access the bucket resources
    * This is done by changing the CORS settings - **Cross Origin Resource Sharing**
        * Set of HTTP headers that the browser can use to determine if the bucket resources allowed to be used by the domain

```bash

# Headers written to JSON file
cat > publicbucketcors.json << EOL
[
    {
      "origin": ["*"], # URLS to be included later for domains
      "responseHeader": ["Content-Type", "Access-Control-Allow-Origin"],
      "method": ["GET", "HEAD"],
      "maxAgeSeconds": 5
    }
]
EOL

gsutil mb -p $PROJECT_NAME -c regional -l $PROJECT_REGION gs://$PUBLIC_ASSETS/
# makes the bucket
gsutil cors set publicbucketcors.json gs://$PUBLIC_ASSETS
# cors command to set the headers to the bucket - allows objects to be fetched by any domain (* in origin)
gsutil iam ch allUsers:objectViewer gs://$PUBLIC_ASSETS
# Makes the bucket from private to public - all users will have object viewing rights
```
`gsutils iam get s://fs2-public-bucket`
This shows all members and their roles  - which will show the allUsers role as storage object viewer

## PubSub Topic and Image subscribing

[pubsub.sh](../../content-google-cloud-engineer-master/common/build/pubsub.sh)

PubSub is a fully managed service - all that happens is you create a topic and publish messages to it. Subsribers to that topic will receive the messages 

`gcloud pubsub topics create $PUB_SUB_TOPIC --project $PROJECT_NAME`  
Once created we can publish messages to it - but currently no one is subsribed so no one is listening

## BigTable
[bigtable.sh](../../content-google-cloud-engineer-master/common/build/bigtable.sh)

In this case BigQuery is used to query BigTable as an external table 

```bash
# This creates a new BigTable instance and cluster - creating a bigtable database for table creation
gcloud beta bigtable instances create $BIGTABLE_INSTANCE_ID \
    --project=$PROJECT_NAME \
    --cluster=$BIGTABLE_CLUSTER_ID \
    --cluster-zone=$PROJECT_ZONE \
    --display-name=$BIGTABLE_DISPLAY_NAME \
    $BIGTABLE_CLUSTER_NUM_NODES_FLAG_VALUE \
    $BIGTABLE_CLUSTER_STORAGE_TYPE_FLAG_VALUE \
    --instance-type=$BIGTABLE_INSTANCE_TYPE # Either dev or prod - dev instance is cheaper
```
### CBT 

CBT is a separate cloud sdk component - which needs to be installed   
`gcloud components list` to see if it's installed, if not you can `gloud components update` and then `gcloud components install cbt`

We then use cbt to create the table and table families (seen in the script)

### Make BigQuery interact with BigTable

BigQuery needs to know how the BigTable data is structured using a table definition file via json

```bash
cat > bigquery_table_def.json << EOL
{
    "sourceFormat": "BIGTABLE",
    "sourceUris": [
        "https://googleapis.com/bigtable/projects/$PROJECT_NAME/instances/$BIGTABLE_INSTANCE_ID/tables/$BIGTABLE_TABLE_ID"
    ],
    "bigtableOptions": {
        "readRowkeyAsString": "true",
        "columnFamilies" : [
            {
                "familyId": "item",
                "onlyReadLatest": "true",
                "type": "STRING"
            },
            {
                "familyId": "buyer",
                "onlyReadLatest": "true",
                "type": "STRING"
            },
            {
                "familyId": "seller",
                "onlyReadLatest": "true",
                "type": "STRING"
            }
        ]
    }
}
EOL
```

### Note: Not all services and functionality is available in all regions! BigTable queries by BigQuery is one of those instances

`gcloud beta bigtable instances list` to list all the bigtable instances open for a project 

## BigQuery

[bigquery.sh](../../content-google-cloud-engineer-master/common/build/bigquery.sh)

```bash
# Make a new bigquery dataset
bq --location=US mk --dataset $PROJECT_NAME:"app_dataset_$ENV_TYPE"

# Make table fro the table definition json file created earlier in the bigtable script
bq mk --external_table_definition=bigquery_table_def.json "app_dataset_$ENV_TYPE.items"
```