# Cloud Load Balancing

Load balancing enables you to distribute load-balanced compute resources in single or multiple regions, meet high availability requirements, scale resource up or down (intelligent autoscaling)

Cloud load balancing allows you to serve content as close as possible to end users with high performance.  
It is a fully distributed software-defined managed service - and is NOT instance/device based - Don't need to manage infrastructure for it

## Types of Cloud Load Balancing

|Internal or external | Load balancer type | Regional or global                    | Supported network tiers | Proxy or pass-through | Traffic type |
|---------------------|--------------------|---------------------------------------|-------------------------|-----------------------|------------- |
| Internal            | Internal TCP/UDP   | Regional                              | Premium only            | Pass-through          | TCP or UDP   |
|                     | Internal HTTP(S)   | Regional                              | Premium only            | Proxy                 | HTTP or HTTPS|
| External            | Network TCP/UDP    | Regional                              | Premium or Standard     | Pass-through          | TCP or UDP   |
|                     | TCP Proxy          | Global in Premium Tier                | Premium or Standard     | Proxy                 | TCP          |
|                     | SSL Proxy          | Effectively regional1 in Standard Tier| Premium or Standard     | Proxy                 | SSL          |
|                     |HTTP(S)             | Effectively regional1 in Standard Tier| Premium or Standard     | Proxy                 | HTTP or HTTPS|

Effectively Regional: Backend Service is global, if standard tier is chosen, the external fowarding rule and external IP address must be regional

## Global vs Regional Load Balancing

* Global Load Balancing - Backends are distributed across multiple regions. Users need access to the same applications and content - provide access by using a single anycast IP address
* Regional Load Balancing - Backends are in one region - only IPv4 termination 

## External vs Internal Load Balancing

* External load balancers - distribute traffic coming from the internet to your Google Cloud Virtual Private Cloud (VPC) network. Global load balancing requires that you use the Premium Tier of Network Service Tiers. For regional load balancing, you can use Standard Tier.

* Internal load balancers - distribute traffic to instances inside of Google Cloud.

External and Internal Load Balancers can walk together to most efficiently distribute load from users: 

![External and Internal Load Balancers](../images/ext_int_lb.PNG =1000x)

## Traffic Type

Load Balancers also need to take into account **type of traffic**:
* **HTTP and HTTPS Traffic:** Handled by external or Internal HTTP(S) Load Balancing
* **TCP Traffic**: Handled by TCP Proxy Load Balancing, Network Load Balancing or Internal TCP/UDP LB
* **UDP traffic:** Handled by Network Load Balancing or Internal TCP/UDP Load Balancing.

## Load Balancing Underlying Technology
* Google Front Ends (GFEs): distributed systems that are located in Google points of presence (PoPs) and perform global load balancing in conjunction with other systems and control planes.
* Andromeda is Google Cloud's software-defined network virtualization stack
* Maglev is a distributed system for Network Load Balancing.
* Envoy proxy is an open source edge and service proxy, designed for cloud-native applications.


| GFE      | Andromeda        | Maglev          | Envoy Proxy     | 
|----------|------------------|-----------------|-----------------| 
| HTTP(S)  | Internal TCP/UDP | Network TCP/UDP | Internal HTTP(S)|
| SSL Proxy| Internal HTTP(S) |                 |                 |
| TCP Proxy|                  |                 |                 |

