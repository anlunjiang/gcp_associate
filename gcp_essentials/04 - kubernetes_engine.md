# Kubernetes Engine

```
Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
```
Googles Kubernetes Engine GKE provides a managed env for deploying, managing and scaling your containerised apps using K8s commands 
The engine consists of multiple nodes grouped to form a **container cluster**  

## Cluster Management
* **Load-Balancing**
* **Node Pools** - designate subsets of nodes within a cluster with the same config
* **Automatic Scaling** - cluster node count
* **Automatic upgrades** - for software
* **Node auto-repair**
* **Logging and monitoring**

### Set compute zone
Approximate location in which your clusters and resources live
> `gcloud config set compute/zone us-central1-a`  

## Creating a K8s Engine Cluster

A cluster consists of at least one master and multiple nodes - The nodes then run K8s processes that enable them to be part of the cluster
> `gcloud container clusters create [CLUSTER-NAME]`

This will take a while before showing an output summarising your running cluster  

|NAME        |LOCATION       |MASTER_VERSION  |MASTER_IP     |MACHINE_TYPE   |NODE_VERSION    |NUM_NODES| STATUS |
|------------|---------------|----------------|--------------|---------------|----------------|---------|--------|
|my-cluster  |us-central1-a  |1.13.11-gke.23  |35.202.244.5  |n1-standard-1  |1.13.11-gke.23  |3        | RUNNING|


### Auth credentials
After creating the cluster - authentication details will need to be retrieved before interacting with the cluster:
> `gcloud container clusters get-credentials [CLUSTER-NAME]` - which returns:

`Fetching cluster endpoint and auth data.`  
`kubeconfig entry generated for my-cluster.`

## Deploying an application to the cluster
The K8s engine uses uses K8s objects to create and manage resources.  
Using the **DEPLOYMENT** for deploying stateless apps like web servers - with **SERVICES** to define rules and expose app on a set of **PODS** as a network service

> `kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0` - creates a deployment representing the hello-world server, with the container image to deploy pulled from the Google Container Repository bucket  
> `kubectl expose deployment hello-server --type=LoadBalancer --port 8080` - Create service that exposes the application to external traffic on a defined port: This service is a LoadBalancer type  
> `kubectl get service` - You can then inspect existing services - with the external ip (may take a bit longer to retrieve ip)

| NAME          |TYPE           |CLUSTER-IP      |EXTERNAL-IP     | PORT(S)        |  AGE |
|---------------|---------------|----------------|----------------|----------------|------|
| hello-server  |LoadBalancer   |10.39.244.114   |35.223.219.203  | 8080:32649/TCP |  63s |
| kubernetes    |ClusterIP      |10.39.240.1     |<none>          | 443/TCP        |  11m |

## Clean up and delete cluster
> `gcloud container clusters delete [CLUSTER-NAME]`