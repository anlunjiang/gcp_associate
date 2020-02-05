# Cloud Shell and gcloud

Google Cloud Shell provides commnad line access to computing resources hosted on GCP. It is Debian based VM with 5GB of home directory persistent memory

> `gcloud compute project-info describe --project <your_project_ID>` - shows all the metadata for your existing project  

```
student_01_9a040eec98ea@cloudshell:~ (qwiklabs-gcp-01-14c83bf72fc8)$ gcloud compute project-info describe --project qwiklabs-gcp-01-14c83bf72fc8
commonInstanceMetadata:
  fingerprint: FIAlw-O3m4U=
  items:
  - key: ssh-keys
    value: student-01-9a040eec98ea:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC/GVwQhPaPsAxtgisaoB/513tzvCUgw/UthjssovHGDhbyLbxQdVd9B1YymcHW6yajfjEFa4Ptr/e6fb750kuMQlhjmJFJXyC4ybteI2b7DXO4TFXGpzQ0JiWjayeAweHlxoI
TTP7MmyTKLyEYZSxtLwclulodTW24Pqz6mPTDqX0IlyBmQsdhjxHdekheS1YdnvKmf7O2jhk70/gJ/8NdvVCdakOX5LH+IP/17v3IsbKSDGUH0b9QvjaXHeOCEhbv/5tpUEy+yWj0z3f+1RJHw1sc6+aNhh+eahM5H158rHk0+u4jJMv5RN0ZZ6LSxWMHdtuqPOiolLoO0yyVc
Ekz
      student-01-9a040eec98ea@qwiklabs.net
  - key: enable-oslogin
    value: 'true'
  - key: google-compute-default-zone
    value: us-central1-a
  - key: google-compute-default-region
    value: us-central1
  kind: compute#metadata
creationTimestamp: '2020-01-29T06:36:35.116-08:00'
defaultNetworkTier: PREMIUM
defaultServiceAccount: 1054749536996-compute@developer.gserviceaccount.com
id: '5401071464509875869'
kind: compute#project
name: qwiklabs-gcp-01-14c83bf72fc8
quotas:
- limit: 1000.0
  metric: SNAPSHOTS
  usage: 0.0
- limit: 5.0
  metric: NETWORKS
  usage: 1.0
- limit: 100.0
  metric: FIREWALLS
  usage: 4.0
- limit: 100.0
  metric: IMAGES
  usage: 0.0
- limit: 8.0
  metric: STATIC_ADDRESSES
  usage: 0.0
- limit: 200.0
  metric: ROUTES
  usage: 22.0
- limit: 15.0
  metric: FORWARDING_RULES
  usage: 0.0
```

## Cloud SDK

gcloud CLI is part of the Cloud SDK - when starting a project from scratch you need to download and install gcloud on your system to use it
> `gcloud init` - This is automatically available in cloud shell though

## Setting Environment Variables

> `export PROJECT_ID=<your_project_ID>`  
> `export ZONE=<your_zone>`

> `echo $PROJECT_ID`  
> `echo $ZONE`

## Create VM in gcloud

> `gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE`  
Note that omitting the  --zone parameter will default to the value set in the describe output

> `gcloud config list [-all]` - List the configurations in your environment  
> `gcloud components list` - List componenets ready to be used  
> `gcloud compute instances describe <your_vm>` - Describe the metadata about your VM (specifications)

> `gcloud compute ssh gcelab2 --zone $ZONE` SSH from CLI to your existing VM  


## Auto-complete and interactive shell

gcloud interactive has auto prompts for commands  - to activate:

> `gcloud components install beta`  
> `gcloud beta interactive` - To enter the interactive shell  
