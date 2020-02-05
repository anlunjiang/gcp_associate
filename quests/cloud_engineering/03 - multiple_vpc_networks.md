# Multiple VPC Networks (Virtual Private Cloud)

![External and Internal Load Balancers](../../images/vpc.PNG =1000x)

* Create custom mode VPC networks with firewall rules
* Create VM instances using Compute Engine
* Explore the connectivity for VM instances across VPC networks
* Create a VM instance with multiple network interfaces

# Create custom mode VPC networks with firewall rules
* Allow for SSH. ICMP (Internet Control Message Protocol) and RDP traffic

For each GCP project there should already be default network with all the possible regions

## Creating a VPC network

1. Using the google console you can use the gui to setup your vpc
2. You can also create vpcs using the terminal  

### On the terminal:

create a vpc network with name privatenet and specify to have a custom subnet
> `gcloud compute networks create privatenet --subnet-mode=custom`

create the us subnet
> `gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24`

create the eu subnet
> `gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20`

you should now have 4 vpc networks established (mynetwork is already made by the qwiklabs session)
> `gcloud compute networks list`

NAME | SUBNET_MODE | BGP_ROUTING_MODE | IPV4_RANGE | GATEWAY_IPV4
-----|-------------|------------------|------------|-------------
default | AUTO | REGIONAL | - | -
managementnet | CUSTOM | REGIONAL | - | -
mynetwork | AUTO | REGIONAL | - | -
privatenet | CUSTOM | REGIONAL | - | -

default and mynetwork are auto mode networks, whereas, managementnet and privatenet are custom mode networks.  
* Auto mode networks create subnets in each region automatically
* Custom mode networks start with no subnets, giving you full control over subnet creation

You can now see all the available VPC sunets that are sorted by VPC network 
> `gcloud compute networks subnets list --sort-by=NETWORK`

NAME | REGION | NETWORK | RANGE
-----|--------|---------|------
default | us-west2 | default | 10.168.0.0/20
default | asia-northeast1 | default | 10.146.0.0/20
default | asia-northeast2 | default | 10.174.0.0/20
default | us-west1 | default | 10.138.0.0/20
default | southamerica-east1 | default | 10.158.0.0/20
default | europe-west6 | default | 10.172.0.0/20
default | europe-west4 | default | 10.164.0.0/20
default | asia-east1 | default | 10.140.0.0/20
default | europe-north1 | default | 10.166.0.0/20
default | asia-southeast1 | default | 10.148.0.0/20
default | us-east4 | default | 10.150.0.0/20
default | europe-west1 | default | 10.132.0.0/20
default | europe-west2 | default | 10.154.0.0/20
default | europe-west3 | default | 10.156.0.0/20
default | australia-southeast1 | default | 10.152.0.0/20
default | asia-south1 | default | 10.160.0.0/20
default | asia-northeast3 | default | 10.178.0.0/20
default | us-east1 | default | 10.142.0.0/20
default | us-central1 | default | 10.128.0.0/20
default | asia-east2 | default | 10.170.0.0/20
default | northamerica-northeast1 | default | 10.162.0.0/20
managementsubnet-us | us-central1 | managementnet | 10.130.0.0/20
mynetwork | us-west2 | mynetwork | 10.168.0.0/20
mynetwork | asia-northeast1 | mynetwork | 10.146.0.0/20
mynetwork | asia-northeast2 | mynetwork | 10.174.0.0/20
mynetwork | us-west1 | mynetwork | 10.138.0.0/20
mynetwork | southamerica-east1 | mynetwork | 10.158.0.0/20
mynetwork | europe-west6 | mynetwork | 10.172.0.0/20
mynetwork | europe-west4 | mynetwork | 10.164.0.0/20
mynetwork | asia-east1 | mynetwork | 10.140.0.0/20
mynetwork | europe-north1 | mynetwork | 10.166.0.0/20
mynetwork | asia-southeast1 | mynetwork | 10.148.0.0/20
mynetwork | us-east4 | mynetwork | 10.150.0.0/20
mynetwork | europe-west1 | mynetwork | 10.132.0.0/20
mynetwork | europe-west2 | mynetwork | 10.154.0.0/20
mynetwork | europe-west3 | mynetwork | 10.156.0.0/20
mynetwork | australia-southeast1 | mynetwork | 10.152.0.0/20
mynetwork | asia-south1 | mynetwork | 10.160.0.0/20
mynetwork | asia-northeast3 | mynetwork | 10.178.0.0/20
mynetwork | us-east1 | mynetwork | 10.142.0.0/20
mynetwork | us-central1 | mynetwork | 10.128.0.0/20
mynetwork | asia-east2 | mynetwork | 10.170.0.0/20
mynetwork | northamerica-northeast1 | mynetwork | 10.162.0.0/20
privatesubnet-eu | europe-west1 | privatenet | 172.20.0.0/20
privatesubnet-us | us-central1 | privatenet | 172.16.0.0/24

default and mynetwork networks have subnets in each region as they are auto mode networks.  
The managementnet and privatenet networks only have the subnets that you created as they are custom mode networks.

### Create the firewall rules for managementnet
This is to allow SSH, ICMP, and RDP traffic to VM instances on the managementnet network.  
Again this can be done of on the console but also on the terminal:
> `gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0`

### Create the firewall rules for privatenet
> `gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

You can now see the firewall rules sorted by VPC network

> `gcloud compute firewall-rules list --sort-by=NETWORK`

NAME | NETWORK | DIRECTION | PRIORITY | ALLOW | DENY | DISABLED
-----|---------|-----------|----------|-------|------|---------
default-allow-icmp | default | INGRESS | 65534 | icmp | - | False
default-allow-internal | default | INGRESS | 65534 | tcp:0-65535,udp:0-65535,icmp | - | False
default-allow-rdp | default | INGRESS | 65534 | tcp:3389 | - | False
default-allow-ssh | default | INGRESS | 65534 | tcp:22 | - | False
managementnet-allow-icmp-ssh-rdp | managementnet | INGRESS | 1000 | tcp:22,tcp:3389,icmp | - | False
mynetwork-allow-icmp | mynetwork | INGRESS | 1000 | icmp | - | False
mynetwork-allow-rdp | mynetwork | INGRESS | 1000 | tcp:3389 | - | False
mynetwork-allow-ssh | mynetwork | INGRESS | 1000 | tcp:22 | - | False
privatenet-allow-icmp-ssh-rdp | privatenet | INGRESS | 1000 | icmp,tcp:22,tcp:3389 | - | False

## Create VM Instances

> `gcloud beta compute instances create managementnet-us-vm --zone=us-central1-c`     
> `--machine-type=f1-micro --subnet=managementsubnet-us --network-tier=PREMIUM`  
> `--maintenance-policy=MIGRATE --service-account=612078084772-compute@developer.gserviceaccount.com`  
> `--image=debian-9-stretch-v20200204 --image-project=debian-cloud --boot-disk-size=10GB`  
> `--boot-disk-type=pd-standard --boot-disk-device-name=managementnet-us-vm --reservation-affinity=any`


> `gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us`

> `gcloud compute instances list --sort-by=ZONE`

NAME | ZONE | MACHINE_TYPE | PREEMPTIBLE | INTERNAL_IP | EXTERNAL_IP | STATUS
-----|------|--------------|-------------|-------------|-------------|-------
mynet-eu-vm | europe-west1-c | n1-standard-1 | - | 10.132.0.2 | 34.77.163.129 | RUNNING
managementnet-us-vm | us-central1-c | f1-micro | - | 10.130.0.2 | 34.67.191.81 | RUNNING
mynet-us-vm | us-central1-c | n1-standard-1 | - | 10.128.0.2 | 34.69.172.82 | RUNNING
privatenet-us-vm | us-central1-c | n1-standard-1 | - | 172.16.0.2 | 35.184.202.229 | RUNNING

There are three instances in us-central1-c and one instance in europe-west1-c.  
However, these instances are spread across three VPC networks (managementnet, mynetwork and privatenet), with no instance in the same zone and network as another

## Connectivity between the VMs

Explore effect of having VM instances in the same zone versus instances in the same VPC

### Ping external IP addresses

SSHing into mynet-us-vm, you can ping any of the other 3 VM Instances just fine regardless of zone or VPC on the EXTERNAL IP ADDRESS

* VM instances should be able to ping any other VM Instance on their external IP address - regardless of zone or VPC
    * Confirms public access controlled by the ICMP firewall rules established earlier 

* VM Instance cannot ping other Instances on internal IP addresses UNLESS they are part of the same VPC network regardless of zone 

VPC networks are by default isolated private networking domains.  
No internal IP address communication is allowed between networks, unless you set up mechanisms such as VPC peering or VPN.

## Create VM instance with multiple network interfaces

Every instance in a VPC network has a default network interface  
You can create additional network interfaces attached to your VMs.  
Multiple network interfaces enable you to create configurations in which an instance connects directly to several VPC networks (up to 8 interfaces, depending on the instance's type)
This depends on your machine type

You can create the Instance on the console and add more network interfaces to the instance

### Explore network interface of the instance

SSHing into the vminstance with the interfaces, you can ping instances by name if they are in the same VPC e.g. 
> `ping -c 3 privatenet-us-vm`

However pinging mynet-eu-vm's internal IP doesnt work:
> `ping -c 4 10.132.0.2`


This is because - every interface gets a specific route for the subnet of that network - with a default of eth0

> `ip route`  
default via 172.16.0.1 dev eth0   
10.128.0.0/20 via 10.128.0.1 dev eth2   
10.128.0.1 dev eth2 scope link   
10.130.0.0/20 via 10.130.0.1 dev eth1  
10.130.0.1 dev eth1 scope link  
172.16.0.0/24 via 172.16.0.1 dev eth0  
172.16.0.1 dev eth0 scope link  

mynet-eu-vm	ip address of 10.132.0.2 isnt included in the raouting table and therefore leave on eth0 the default route - which obviously isn't on the same VPC network so therefore cannot be pinged