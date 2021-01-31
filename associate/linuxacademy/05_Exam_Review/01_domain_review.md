# Domain Objective Review

[[TOC]]

# Section 1 - Setting up a cloud solution environment
## 1.1 - Cloud projects and accounts
* Creating projects
    * Since everything in GCP is associated with a project, knowing how to create a project is a very important.
    * Can create using the UI or the CLI
* Assigning users to pre-defined IAM roles within a project
    * Roles can be very granular - users only need the role required for their task and nothing more 
* Linking users to G Suite identities
    * Google IAM is not an identity provider it just sets roles for users
    * Google identity is an identity provider as well as GSuite
    * Also allows Gmail and group accounts to serve as users
* Enabling APIs within Projects
    * GCP is just a set of REST APIs - if you want a service, usually you just enable the API for it via cli or ui
* Provisioning one or more Stackdriver accounts
    * Can create a single stackdriver account that monitors multiple projects
    * If you just have one project you can create an account in the same project you're monitoring 
    * else create a separate stack driver project

## 1.2 Managing Billing configuration

Linking a project to a billing account is how you pay for the resources in the project

* Creating one or more billing accounts 
* Linking projects to a billing accont
* Establishing billing budgets and alerts
    * Used to catch intensive tasks that might not have been expected - e.g. overusing a GPU in a compute engine VM
* Setting up billing exports to estimate daily/monthly charges
    * Can be used for analysis to help cut current costs and predict future costs more accurately 
    * Can export logs into say bigquery

## 1.3 - Installing and configuring the SDK

# Section 2 - Planning and configuring a cloud solution

## 2.1 Planning and estimating GCP product use using the pricing calculator
Can help determine if the solution proposed is affordable before we even setup a single resource

## 2.2 Planning and configuring compute resources
Will ask questions on the best way to do something - and to recognise if something isnt designed well

* Selecting appropriate compute choices for a given workload (CEngine, KEgine, AEngine)
* Using preemptible VMs and custom machine types as appropriate

## 2.3 Planning and configuring data storage options
* Products choice (CloudSQL, Bigquery, CloudSpanner, Cloud bigtable)
* Storage options * Regional, Multi-regional, nearline, coldline

## 2.4 Planning and configuring network resources
* Differentiating load balance options
* Identifying resource locations in a network for availability
* Configure Cloud DNS

# Section 3 - Deploying and implementing a cloud solution

## 3.1 Deploy and implement Cloud Engine resources
* Launching a compute instance using Cloud Console and Cloud SDK (gcloud) (assign disks, availability policy, SSH keys)
* Creating an autoscaled managed instance group using an instance template 
* Generating/uploading custom SSH key for instances
* Config VM for stackdriver monitoring and logging
* Assessing compute quotas and requesting increases
    * Can request an increase on the limits from the Google Cloud support team
* Installing Stackdriver Agent for monitoring and logging 
    * For linux you can bootstrap those agents for the VMs, no automation for windows though

## 3.2 Deploy and implement Kubernetes Engine resources
Google manages all the complexity of running an actual K8s cluster  
Can create clusters using REST API, Cloud SDK or UI - NOT kubectl command! Kubectl is to deploy the resources (such as pods) to the cluster. Pods are just one or more containers that you want to run together on the same node. The containers inside a pod can communicate with each other via internal ip (localhost)

* Deploying a K8s cluster
* Deploy a container application to K8s engine using pods
* Configure L8s application monitoring and logging

## 3.3 Deploy and implement app engine and cloud function
App Engine is Google's fully managed application platform - great for static web pages. App engine regions CANNOT be changed, applications cannot be deleted only disabled from a project, easy to roll back broken changes in versions  
It is a serverless service - good for lightweight backends that responds to http events
* Deploy an application to App Engine ( scaling config, versions, traffic splitting)
* Deploy a cloud function that receives GCP events (Pub/Sub, Cloud storage objects)

## 3.4 Deploy and implement data solutions
* Initialising data systems with products (Cloud SQL, Bigquery, Cloud Datastore, Cloud Spanner, Pub/Sub, Bigtable, Cloud dataproc, Cloud Storage)
* Loading data (CLI upload, API transfer, import/export, load data from cloud storage, streaming with Pub/Sub)
* Cloud Storage - Parallel uploads, breaks a large file down to much smaller files and uploads in parallel, making it much faster if you have the bandwidth

## 3.5 Deploy and implement network resources
* Creating a VPC with subnets
* Launching a CE instance with custom network config (internal only IP, private access, static external and internal IP, network tags etc)
* Creating ingress and egress firewall rules for a VPC
* Creating a VPN between a VPC and an external networ using Cloud VPN
* Creating a load balancer to distribute application network traffic to an application (e.g. Global HTTPs load balancer etc)

## 3.6 Deploy a solution using Cloud Launcher
easy to launch apps such as wordpress with cloud launcher
* Browsing the cloud launcher catalog 
* Deploy a cloud launcher marketplace solution

## 3.7 Deploy an application using deployment manager
Google's infrastructure as code solution
* Developing deployment manager templates to automate deployment
* Launching a deployment manager template to provision GCP resources and configure an application automatically  

# Section 4 - Ensuring successful operation of a cloud Solution

## 4.1 Manage CE resources
* Manage a single VM instance (start/stop, edit config, delete and instance)
* SSH/RDP into an instance
* Attch a GPU to a new instance and installing CUDA libraries ( not all GPUs are in all zones)
* View current running VM inventory (instance id etc)
* Work with snapshots (create a snapshot from a VM, view, and delete snapshots)
* Working with images
* Working with instance groups ( Set auto scale parameters, assign/create instance template, remove instance group
* Work with management interfaces, SDK, Cloud Shell, Cloud Console

## 4.2  4.2 Manage K8s resources
* View current cluster list (nodes, pods, services)
* Browse container image repo and view container image detials
* Working with nodes (add/edit delete)
* Pods
* Services

## 4.3 Managing App Engine resources
* Adjust application traffic splitting parameters
* Setting scaling params for autoscaling instances
    * min idle instances - always ready for a traffic surge
    * max idle instances - make sure you dont spend too much 
    * Can tune to your scaling needs 

## 4.4 Manage data solutions
* Executing queries to retrieve data from data instances
* Estimating costs of a bq query ( using the --dryrun flag)
    * bq charged based on the data you read, not the data you return (limit would not do anything, it happens after you read the data/)
* Back up and restore data instances
* Review job status in cloud dataproc and bq
* Moving objects between cloud storage buckets
* setting up object lifecycle management policies for cloud storage buckets 

## 4.5 Managing networking resources
* Adding a subnet to an existing VPC
* Expanding a CIDR block subnet to have more IP address, as long as there are no overlaps
* Reserving Static external IP address

## 4.6 Monitorin and logging 
Not expected to know in a really low level
* Create stackdriver alert based on resource metrics
* Custom metric
* Log sinks to export logs to external sytstems (on prem or to bq)
* Viewing and filtering logs and details in Stackdriver

# Section 5 - Configuring access and security

## 5.1 Manage IAM
* View account IAM, assign roles and define custom roles

## 5.2 Managing service accounts
* Managing service accounts with limited scopes
* Assigning a service account to a VM instance
* granting access to a service account in another project

## 5.3 Viewing audit logs for project and managed services