# Application Deployment

[[TOC]]

# Configuring Common Services

This section will follow on the linux academy codebase found in this repo as `content-google-cloud-engineer-master` (cgcem)

## Enable APIs

[enableapis.sh](../../content-google-cloud-engineer-master/common/build/enableapis.sh)

> `gcloud services enable <api_name_id>` e.g. storage-component.googleapis.com

## Network setup

[network.sh](../../content-google-cloud-engineer-master/common/build/network.sh)

Creates a VPC with two subnets, one for the products and one for the ads app

> `gcloud compute networks create <network_name> --project=<proj_name> --subnet-mode=custom`

```bash
gcloud beta compute networks subnets create <subnet_name>  
--project=$PROJECT_NAME   
--network=$SERVICES_NETWORK \  
--region=us-central1 \
--range=10.29.0.0/24 \ # CIDR Notation Classless Interdomain Routing
--enable-private-ip-google-access \
--enable-flow-logs
```
* flow logs will enable us to log the flow of traffic in the network
    * Great for debugging and analysis
    * Can be expensive to store since there are a lot of logs
* This creates a subnet with a custom defined ip range
* It then creates firewall rules to allow internal traffic on network to flow

> `gcloud compute networks (subnets) list` View all created (sub)networks in the project


