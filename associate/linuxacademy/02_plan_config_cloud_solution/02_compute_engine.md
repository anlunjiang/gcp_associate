
# Compute Engine

[[TOC]]

* All about Virtual Machines VMs
* Optimal for ultra high performance workloads (GPUs and Local SSDs, 3D rendering)
* Also good for content management systems, databases and non containerised apps

## Creating an Instance
* Select name unique to the project
* Select the region and the zone
* Machine Type - Hardware config, cpu, ram etc
    * This can be customised (number of cpu, ram, gpu)
    * This will affect the cost of running the vm
* Select OS for boot disk
    * Can use sql server images
    * custom images
    * snapshot, point in time backup of some imae
* Select a service account - creds that your code and tool use to interact with other services
* Config firewall (HTTP, HTTPS traffic)
* Automation - startup scripts to bootstrap code or configure
* Choose if pre-emptive instance
    * Shortlived (24hr max) to utilise excess compute capacity at a discounted rate (80% off)
    * Will be turned off however if the capacity is needed
    * Good for bulk processing, can help but wont stop the process if is lost
    * Great for STATELESS WORKLOADS

## SSH

After your VM instance is created, you cant then connect into the terminal of the instance via ssh.
With windows you need to RDP protocol to access the server (chrome RDP plugin works)
* With windows RDP you need to setup a password in the console
    * after you enter the password you can now control the windows server like you would any windows machine

