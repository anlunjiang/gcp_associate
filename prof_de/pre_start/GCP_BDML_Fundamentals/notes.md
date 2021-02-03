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
    
