# Create and Manage Cloud Resources: Challenge Lab

* Create all resources in the default region or zone, unless otherwise directed.
* Naming normally uses the format team-resource; for example, an instance could be named nucleus-webserver1.
* Allocate cost-effective resource size. This is the guidance the monitoring team is willing to share: unless directed, use f1-micro for small Linux VMs, and use n1-standard-1 for Windows or other applications, such as Kubernetes nodes.

## Topics tested:

* Create an instance
* Create a 3-node Kubernetes cluster and run a simple service
* Create an HTTP(s) load balancer in front of two web servers

## Task 1: Create a project jumphost instance

You will use this instance to perform maintenance for the project.  

Requirements:
* Name the instance nucleus-jumphost.
* Use an f1-micro machine type.
* Use the default image type (Debian Linux).

## Task 2: Create a Kubernetes service cluster

The team is building an application that will use a service running on Kubernetes. You need to:  

* Create a cluster (in the us-east1-b zone) to host the service.
* Use the Docker container hello-app (gcr.io/google-samples/hello-app:2.0) as a place holder; the team will replace the container with their own work later.
* Expose the app on port 8080.

> `gcloud config set compute/zone us-east1-b`
> `gcloud container clusters create [CLUSTER-NAME]`
> `gcloud container clusters get-credentials [CLUSTER-NAME]`
> `kubectl create deployment nucleus-service --image=gcr.io/google-samples/hello-app:2.0`
> `kubectl expose deployment nucleus-service --type=LoadBalancer --port 8080`
> `kubectl get service` 

## Task 3: Set up an HTTP load balancer
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant.  
* Create an HTTP load balancer with a managed instance group of 2 nginx web servers. 
* Use the following code to configure the web servers; the team will replace this with their own configuration later.

```bash 
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

You need to:  
* Create an instance template.
> `gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh`

* Create a target pool.
> `gcloud compute target-pools create nginx-pool`

* Create a managed instance group.
> `gcloud compute instance-groups managed create nginx-group \`    
> `--base-instance-name nginx \`    
> `--size 2 \`  
> `--template nginx-template \ `  
> `--target-pool nginx-pool`

* Create a firewall rule to allow traffic (80/tcp).
> `gcloud compute firewall-rules create firewall --allow tcp:80`

* Create a health check.
> `gcloud compute http-health-checks create http-basic-check`

* Create a backend service, and attach the managed instance group.
> `gcloud compute backend-services create nginx-backend --protocol HTTP --http-health-checks http-basic-check --global`

> `gcloud compute backend-services add-backend nginx-backend \`  
      > `--instance-group nginx-group \`  
      > `--instance-group-zone us-east1-b \`  
      > `--global`

* Create a URL map, and target the HTTP proxy to route requests to your URL map.
> `gcloud compute url-maps create web-map --default-service nginx-backend`

> `gcloud compute target-http-proxies create http-lb-proxy --url-map web-map`

* Create a forwarding rule.
> `gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80`  
> `gcloud compute forwarding-rules create http-content-rule --global --target-http-proxy http-lb-proxy --ports 80`  