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
> `gcloud compute instances list --sort-by=ZONE` list all instances
> `gcloud compute instances describe <your_vm>` - Describe the metadata about your VM (specifications)  
> `gcloud compute firewall-rules create [rule_name] --network=[vpc_name] --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`
> `gcloud compute firewall-rules list --sort-by=NETWORK`
> `gcloud compute networks create [vpc_name] --project=[proj_name] --subnet-mode=custom --bgp-routing-mode=regional`
> `gcloud compute networks subnets create [subnet_name] --project=[proj_name] --range=10.130.0.0/20 --network=managementnet --region=us-central1`
> `gcloud compute networks list` - list all the vpcs in your project
> `gcloud compute networks subnets list --sort-by=NETWORK` - list all subnets
> `gcloud compute ssh gcelab2 --zone [YOUR_ZONE]` - SSH into your linux VM instances from cli

> `gcloud components list` - List componenets ready to be used  
> `gcloud components install beta` - Install interactive gcloud beta  

> `gcloud beta interactive` - To enter the interactive shell  

> `gcloud sql connect [name of instance] --user=root` - connect to an existing cloud sql (can be mysql, sqlserver etc) instance with user as root

> `gsutil ls gs://[bucket_name]` - Look at the contents of a bucket


