# Network and HTTP Load Balancers

Load balancing enables you to distribute load-balanced compute resources in single or multiple regions, meet high availability requirements, scale resource up or down (intelligent autoscaling)

## Create multiple web server instances

This is to simulate a cluster of machines using Nginx web servers - serving static content
This uses Instance Templates and Managed Instance Groups:
* Instance Templates 
    * A Global resource used to create VM Instance and managed instance groups
    * They define the machine type, boot disk image, container image, label etc
    * These instances all usually have identical configs
* Managed Instance Groups 
    * Using Instance templates it instantiates a number of VM instances

To create the web server cluster:
1. Create startup script to be used by every VM instance to setup Nginx server upon startup
2. create an instance template to use the startup script
3. create a target pool - allows single access point to all instances in a group and is necessary for LB -associated with a region
4. Create a managed instance group using the instance template 

---
1. Startup Script
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
2. Instance Template 
> `gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh`
3. Target Pool
> `gcloud compute target-pools create nginx-pool`
4. Managed Instance Group
> `gcloud compute instance-groups managed create nginx-group \`    
> `--base-instance-name nginx \`    
> `--size 2 \`  
> `--template nginx-template \ `  
> `--target-pool nginx-pool`

NAME | LOCATION | SCOPE | BASE_INSTANCE_NAME | SIZE | TARGET_SIZE | INSTANCE_TEMPLATE | AUTOSCALED
-----|----------|-------|--------------------|------|-------------|-------------------|-----------
nginx-group | us-central1-a | zone | nginx | 0 | 2 | nginx-template | no

You should also now see 2 VM instances created on the dashboard or if you type:
> `gcloud compute instances list`  

NAME | ZONE | MACHINE_TYPE | PREEMPTIBLE | INTERNAL_IP | EXTERNAL_IP | STATUS
-----|------|--------------|-------------|-------------|-------------|-------
nginx-52ms | us-central1-a | n1-standard-1 || 10.128.0.2 | 34.68.193.250 | RUNNING
nginx-j3fm | us-central1-a | n1-standard-1 || 10.128.0.3 | 35.192.157.210 | RUNNING

### Configure Firewall 

To connect to the machines on port 80 via the external ip address...

> `gcloud compute firewall-rules create firewall --allow tcp:80`

## Create a Network Load Balancer

Network load balancing balances the load based on incoming IP data, such as address, port, and protocol type.  
You also get some options that are not available with HTTP(S) load balancing: Load balance extra TCP/UDP-based protocols such as SMTP traffic.   
Network load balancing allows your app to inspect the packets, where HTTP(S) load balancing does not.

* Create a network LB targeting instance group
> `gcloud compute forwarding-rules create nginx-lb \`  
> `--region us-central1 \`  
> `--ports=80 \`  
> `--target-pool nginx-pool`  

* List all Google Compute Engine forwarding rules in your project  

NAME | REGION | IP_ADDRESS | IP_PROTOCOL | TARGET
-----|--------|------------|-------------|-------
nginx-lb | us-central1 | 35.225.187.111 | TCP | us-central1/targetPools/nginx-pool

 You can enter the lb from your browser on the ip address - that points to the webservers

 ## Create a HTTP(s) Load Balancer

 HTTP(S) load balancing provides global load balancing for HTTP(S) requests destined for your instances.  
 You can configure URL rules that route some URLs to one set of instances and route other URLs to other instances.  
 Requests are always routed to the instance group that is closest to the user, provided that group has enough capacity and is appropriate for the request.  
 If the closest group does not have enough capacity, the request is sent to the closest group that does have capacity.

### Health Check

* Before creating the LB - check that the instance is responding to HTTP/HTTPS traffic
    * > `gcloud compute http-health-checks create http-basic-check`

* Define HTTP Service, map port name to the relevant port for the instance group so the LB service can forward traffic to named port
    * > `gcloud compute instance-groups managed \`  
      > `       set-named-ports nginx-group \`  
      > `       --named-ports http:80`  

* Create backend Service
    * > `gcloud compute backend-services create nginx-backend \`  
      > `      --protocol HTTP --http-health-checks http-basic-check --global`  
    * Backends contain config values for GCP LB services
    * For this the backend uses the basic health check we created earlier as its health check

* Add the instance group to the created backend service
    * > `gcloud compute backend-services add-backend nginx-backend \`  
      > `--instance-group nginx-group \`  
      > `--instance-group-zone us-central1-a \`  
      > `--global`  

* Create a default URL map that will direct all incoming requests to all your instances
    * > `gcloud compute url-maps create web-map \`  
      > `    --default-service nginx-backend`
    * Uses the created backend as the default-service

* Create target HTTP proxy server to route requests to your URL map
    * > `gcloud compute target-http-proxies create http-lb-proxy \`  
      > `--url-map web-map`

* Create a global forwarding rule to handle and route incoming requests. 
    * > `gcloud compute forwarding-rules create http-content-rule \`  
      > `        --global \`  
      > `        --target-http-proxy http-lb-proxy \`  
      > `        --ports 80`  
    * A forwarding rule sends traffic to a specific target HTTP or HTTPS proxy depending on the IP address, IP protocol, and port specified - does not support multiple ports

> `gcloud compute forwarding-rules list`

NAME | REGION | IP_ADDRESS | IP_PROTOCOL | TARGET
-----|--------|------------|-------------|-------
http-content-rule || 34.102.209.74 | TCP | http-lb-proxy
nginx-lb | us-central1 | 35.225.187.111 | TCP | us-central1/targetPools/nginx-pool

Overview:
* User Accesses forwarding rule http-content-rule ip address 34.102.209.74 at port 80
* http-content-rule points to a proxy (of that ip/port) which operates a url map that points to your backend service
* Backend will point you to the instances but also perform a health HTTP response check on the backend

This image will make more sense:

![External and Internal Load Balancers](../../images/http_lb.PNG =1000x)

* A forwarding rule directs incoming requests to a target HTTP proxy.
    * Forwarding rules route traffic by IP address, port and protocol to a LB config of target proxy, URL map and one or more backend service
    * Each forwarding rule provide a single IP address, can only reference 80, 8080 ports for HTTP and 443 for HTTPS 
* The target HTTP proxy checks each request against a URL map to determine the appropriate backend service for the request.
    * One or more forwarding rules direct traffic to the target proxy, and the target proxy consults the URL map to determine how to route traffic to backends
    * URL maps direct incoming requests to backend services
* The backend service directs each request to an appropriate backend (instance) based on serving capacity, zone, and instance health of its attached backends  
    * A  LB can have multiple backend services to direct incoming traffic to more attached backends (Instance groups)