# Regions and Zones

* **Region** - Specific geographical location where you can host your resoures

* **Zone** - Each region has one or more zones e..g US West coast has us-west1-a, us-west1-b and us-west1-c

Resources that live in a zone (VM, persistent disks) are *Zonal Resources* where as static external IP addresses are regional

Regional resources can be used by any resources in that region (any zone in that region)  
Whereas zonal resource can only be used by resources in that zone

e.g.  
* To attach a persistent disk to a VM Instance - both resources must be in the same zone   
* To attach a static IP to an instance, that instance must be in the same region

Segregating resources like this provides isolation from physical and software service failures, leading to failure independence and robust systems

Some resources, like images are global and can be used regardless of region or zone

## Zones and Clusters

* Clusters: distinct physical infrastructure, power, cooling, network and security

Each zone is hosted in one or more clusters and the GCP compute engine maps zones to clusters for each organisation

Therefore the **us-central1-a** zone might be hosted in different clusters in different organisations  
This is beneficial due to:
* Resources are balanced across clusters in a region
* List of zones to choose is manageable despite adding more clusters

Link for list of locations and there region/zone codes: [regions and zones](https://cloud.google.com/compute/docs/regions-zones#available)

## Choosing a region and zone

You choose your regions and zones depending on several reasons:

* **Failure handling:** Distributed resources across multiple zones and regions to be fault tolerant. Traffic can be diverted from one zone to another in the event of downtime
* **Latency:** Choosing regions and zones that are close to points of service 
* Zones have high bandwidth low latency connections to zones of the same region

## Maintenance

 By default, all VM instances are set to
 * Live migrate: Compute engine automatically migrates your running instance  - performance may suffer but your service remains online 

You can choose to set it yo reboot or terminate though in this event

## Notes

* Communication within and regions is usually always cheaper that across regions
* Important systems should have redundancy in multiple zones