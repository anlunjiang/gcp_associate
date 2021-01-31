# Kubernetes Cluster Architecture

Within the GKE - a cluster constists of

* At least one cluster master
* Multiple worker nodes

These machines run the K8s cluster orchestration system

## Cluster Master
Runs the K8s control processes - API  server, scheduler and core resource controllers. GKE will upgrade the K8s version on the master automatically (or manually)

Master is the unified endpoint of your cluster - all interactions with the cluster are done via K8s api calls, and the master runs the K8s API server to handle the calls  
Three ways to make these K8s calls:

* Directly wia HTTP/gRPC
* K8s command-line client `kubectl`
* Interacting with the UI in cloud console

The API server hosted on the master is the single 'source of truth'

### gcr.io (google cloud repository) 
When creating or updating a cluster, container images for K8s are pulled from the gcr registory. 

## Nodes
A cluster usually has at least one node - managed by the master. Nodes run the services to support docker containers that make up the workloads. These services include the docker runtime and the K8s node agent (kubelet) which communicate with the master  - starting and running docker containers on that node   
You can inspect a node#s properties:
> `kubectl describe node [NODE_NAME] | grep Allocatable -B 4 -A 3` returns Capacity and Allocatable fields for ephemoral storage, memory and CPU

When creating clusters you can also specify more parameters such as number of nodes, node replication factor, zones etc (more detail in K8s orchestration md)
> `gcloud container clusters create example-cluster \`  
> `--zone us-central1-a \`  
> `--node-locations us-central1-a,us-central1-b,us-central1-f \`  
> `--num-nodes 2 --enable-autoscaling --min-nodes 1 --max-nodes 4`  

Creates an autoscaling multi-zonal cluster with six nodes across three zones, with a minimum of one node per zone and a maximum of four nodes per zone