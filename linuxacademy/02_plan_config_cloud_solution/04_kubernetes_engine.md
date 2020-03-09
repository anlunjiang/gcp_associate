# Kubernetes Engine

[[TOC]]

K8s is the leading container orchestration platform
It is like a giant server made up of lots of smaller servers - and these servers run pods

# Pods

Pods are one or more containers that should be run and scaled together as a single service 
* Each pod has its own IP address that is used to interact with other pods
* K8s has controllers that manage pods for us:
    * Deployment.yaml: Desired State Configuration
        * How many instances of a pod we cant to run on the cluster 
        * K8s ensures that those pods are always up and running
    * DaemonSet.yaml: 
        * Ensures a copy of a pod runs on each node in a cluster 
        * Good for monitoring
* Pods are ephemoral and can be replaced with different ip addresses
    * We need services  - to say connect backend and frontend pods together without thinking about individual pods
    * Services act as an entry point and load balancer for our applications 

# Kubernetes Engine

A google managed way to run a Kubernetes Cluster: K8s is not just one thing, it is several components that all work together to form kubernetes

## When to use K8s Engine

There are many reasons to use containers:

    1. High level of app portability
    2. dev lapotp and prod server discrepencies are almost none
    3. Helps migrate from on prem to cloud

* You need ensure that your apps are running when you expect them to in containerised workloads - using K8s
* K8s Engine removes the maintenance and activities keeping a K8s cluster up and running 

When setting up a K8s cluster you can choose if the master and nodes are all in one zone, or multi-regional. There is a tradeoff between availability and price here to depends on the application
* Like creating VM Instances, you need to select the machine types, node image (container optimised OS by Google) and Cluster Version, node size
    * Automatic Node upgades - ensure nodes are always running the latest version of K8s
    * Automatic Node Repair - Unhealthy nodes auto replaced

* K8s allows us to think of a collection of nodes as containerised apps

# Deploy

You can then deploy your cluster with a container image (more than one is allowed)
* you can set env variables 
* Sends .yaml files to the k8s rest api

You can then create a service by clicking expose: which will open a port and protocol associated with it:
* Cluster IP: Service will have an internal IP address that other services can use but not public
* Node port: Service listens on a specific port, same port on each selected node
* Load balancer: Creates an external IP address and distributes traffic to the pods 

# Best Practices
Separating code from configuration
K8s has something called config maps:
* Config maps: Allows us to take configuration data and pass it into the pods 
    * e.g. if you have an app that uses cloud datastore or s3, you don't want to hard code it, but pull it from an env var instead without changing the code
    * config map would have a key value for s3 and cloud storage and pass it to container
    * NOT for PASSWORDS and sensitive info 
* Secrets: good for sensitive information 
    * Similar to config maps except the information is encrypted
    * create a secret in the Kubectl cli, then reference that secret in yaml
   