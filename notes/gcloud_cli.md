# gcloud command line tool overview

This is the primary CLI to GCP - Used to create and manage 
* VM Instances
* SQL Instances
* Kubernetes Engine Cluster
* Dataproc clusters and jobs
* DNS managed zones and record sets
* Deployment manager deployments

> `gcloud auth list` - list active account name  
> `gcloud config list project` - list the active project ID  
> `gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone [your_zone]` - create a new vm instance  
> `gcloud compute instances create --help ` - will point out all the defaults when creating a VM instance  
> `gcloud config set compute/zone` - bypass having to set --zone each time  
> `gcloud config set compute/region` - bypass having to set region
> 