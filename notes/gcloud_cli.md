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
> `gcloud config list [-all]` - List the configurations in your environment  
> `gcloud config set compute/zone` - bypass having to set --zone each time  
> `gcloud config set compute/region` - bypass having to set region  

> `gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone [your_zone]` - create a new vm instance  
> `gcloud compute instances create --help ` - will point out all the defaults when creating a VM instance  
> `gcloud compute instances get-serial-port-output instance-1 --zone us-central1-a` - see if a windows server vm is ready for RDP  
> `gcloud compute instances describe <your_vm>` - Describe the metadata about your VM (specifications)  
> `gcloud compute ssh gcelab2 --zone [YOUR_ZONE]` - SSH into your linux VM instances from cli

> `gcloud components list` - List componenets ready to be used  
> `gcloud components install beta` - Install interactive gcloud beta  

> `gcloud beta interactive` - To enter the interactive shell  
