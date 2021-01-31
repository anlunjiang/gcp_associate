# Planning and Configuring Compute Resources

[[TOC]]

# Comparing Compute Options
3 Core pillars of any cloud provider
1. Networking
1. Compute
1. Storage


* App Engine - Fully managed application platform - deploy web apps and mobile backends
    * web based applications
    * High availability
    * App portability is NOT a concern
    * HTTP applications
* Cloud Function - Fully managed service - eventy triggered Node.js code
    * Able to respodnd to events that are triggered by different cloud services 
    * Code to execute in response to cloud events (limits 540 second duration)
    * Javascript
* Kubernetes Engine - containerised workloads, app focused 
    * Running containerised workloads
    * Application portability is important 
    * focus on container as unit deployment 
* Compute Engine - VM service - all workloads, OS level control and high performance available 
    * Create, scale and load balance traffic 
    * need control over OS, CPU, GPU, SSD, RAM etc
    * lift and shift an existing application 
    * workload is NOT containerised (otherwise use K8s engine)
    * Performing bulk processing and require a pre-emptible instances
        * Pre-emptible instance - temporary isntance you can have to add some temporary compute power 
