# Networking Options 

[[TOC]]

* Virtual Private Clouds
* Load Balancing
* Cloud DNS

# Virtual Private Clouds (VPCs)
VPCs are just network - every project has a default network, but can also create additional ones
* VPCs are a global resource
    * They are a container for smaller sub-networks (sub-nets)
    * Sub-nets will always have an ip-address range
        * Within a VPN sub-net ip ranges cannot overlap
        * Inside a subnet all ip-addresses need to be unique

![](../../images/vpc-subnets.PNG =900x)

## Two types of VPC
* Automode - Automatically creates subnets for you in each region (since subnets are a regional resource)
    * Google will create the r31nges for you
* Custom - custom creation of subnets and ip ranges
    * Good for migrating from existing infrastructure and keeping configuration of it

Say you have on-prem application to migrate to GCP, without editing config, the app uses a component that requires a 3rd party license on a server with ip address 192.168.1.20

* You can then create a custom VPC with a subnet: 192.168.1.0/24
* Create a compute engine VM Instance within that subnet and set the internal ip address to 192.168.1.20

## Traffic inside of a network

 * Networks are configured to allow instances within a VPC to send traffic to each other even across sub-nets on their private ip
 * External IP addresses work in the same way and can be exposed to public, with firewall options 

 # Load Balancing 

Load balancing enables you to distribute load-balanced compute resources in single or multiple regions, meet high availability requirements, scale resource up or down (intelligent autoscaling)

Cloud load balancing allows you to serve content as close as possible to end users with high performance.  
It is a fully distributed software-defined managed service - and is NOT instance/device based - Don't need to manage infrastructure for it

Support HTTP/S, TCP, SSL and UDP
## Choosing a Load Balancer

![](../../images/lb.PNG =900x)

# Cloud DNS

Google Cloud DNS (Domain Name System) service - maps requests for domian names into IP addresses
Low Latency and High Availability