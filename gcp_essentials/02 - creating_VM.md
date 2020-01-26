# Creating a Virtual Machine on GCP

GCP allows VM creation for Linux and Windows Server

## Regions and zones

Resources can live in regions or zones. 
* Regions - specific geographic location where resources run
* Each regions has one or more zones 

e.g. The United States region has zones for us-central1-a, us-central1-b etc

Zonal resources live in a zone, the VM instances and persistent disks all live in that zone 

Attaching an instance to disk requires **both resources to be in the same**, same with static IP address to an instance

## Creating a VM Instance
There are two main ways of creating VMs on GCP:
1. CLI on cloud shell 
2. Console

Either way there are parameters you must choose that define the properties of the VM that will be created:

* Name: gcelab
* Region: us-central1 (Iowa)
* Zone: us-central1-c
* Machine Type: 2vCPUs (n1-standard-2) 7.5GB RAM instance 
* Boot Disk: New 10GB standard persistent disk OS Image Debian GBU Linux 9 
* Firewall: Allow HTTP traffic (for web servers)
  
You can then SSH into your VM

## Install NGINX webserver on your VM

>` sudo su -`  
>` apt-get update`  
>`apt-get install nginx -y`  
>`ps auwx  | grep nginx`  to check if the process is running

You can then see the webpage by going to the external IP address

## Creating VMs on CLI

> `gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone [your_zone]`

The instance created has these default values:

* The latest Debian 9 (stretch) image.
* The n1-standard-2 machine type. In this lab you can select one of these other machine types if you'd like: n1-highmem-4 or n1-highcpu-4. When you're working on a project outside of Qwiklabs, you can also specify a custom machine type.
* A root persistent disk with the same name as the instance; the disk is automatically attached to the instance.

> `gcloud compute instances create --help ` will point out all the defaults

Note: You can set the default region and zones that gcloud uses if you are always working within one region/zone and you don't want to append the --zone flag every time. Do this by running these commands :

> `gcloud config set compute/zone`  
> `gcloud config set compute/region`

## SSH via CLI

> `gcloud compute ssh gcelab2 --zone [YOUR_ZONE]`

## Windows server VM
Instead of Linux, you can create a windows server VM instance by changing boot disk to Windows Server 2012 R2 Datacenter    

You can then RDP (Remote desktop protocol) into the server once you run 
> `gcloud compute instances get-serial-port-output instance-1 --zone us-central1-a`

and `Finished running startup scripts` is the output

Once done you can then RDP in using chrome RDP extension using the credentials you set when setting up RDP configuration
