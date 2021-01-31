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

> `gcloud container clusters create [CLUSTER-NAME]` - Create a K8s cluster
# IMPORTANT BELOW
> `gcloud container clusters get-credentials [CLUSTER-NAME]` - After creating the cluster - authentication details will need to be retrieved before interacting with the cluster 
# IMPROTANT ABOVE
> `gcloud beta interactive` - To enter the interactive shell  

> `gcloud deployment-manager deployments create advanced-configuration --config nodejs.yaml` create a deployment from a dep manager

> `gcloud sql connect [name of instance] --user=root` - connect to an existing cloud sql (can be mysql, sqlserver etc) instance with user as root

> `gsutil ls gs://[bucket_name]` - Look at the contents of a bucket

> `kubectl apply -f services/hello-green.yaml` update a service identified in the yaml for a new version
> `kubectl create -f deployments/auth.yaml` create a deployment from auth yaml file
> `kubectl edit deployment <dep_name>` edit a yaml of the deployment by name - this will update the deployment and ebgin a rolling update
> `kubectl get deployments` list deployments
> `kubectl get replicasets` get list of replicasets
> `kubectl get pods` list pods
> `kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'` look at specific features of pods
> `kubectl rollout history deployment/hello` look at rollout update history
> `kubectl rollout pause/resume/undo deployment/hello` pause, resume or undo an update to a deployment
> `kubectl scale deployment <dep name> --replicas=3` scale the number of replicas within a deployment to a number

n.b. containers --> pods --> replicasset --> deployments --> service
  * Use lss containers/replicaset if you want to use a canary deployment (less chance of being served new version)
  * you can use sessionAffinity within the service yaml to bind to say IP for consistent user experience
  * You can use two deployment version ( blue green deployment) and apply a new yaml pointing to the same service