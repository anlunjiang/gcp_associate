# Instance Groups

[[TOC]]

Instance Groups allow you to think about a group of isntances as a single entity. There are two types of instance group:
* Managed
* Unmanaged

# Unmanaged instance groups
Do not require the use of a template. We can use existing instances with different configurations to create a group and load balance. 
* This should only be used if you cannot use managed instance groups
* Unmanged instance groups cannot be autoscaled

# Managed Instance Groups
Allow us to create a template for an instance and then autoscale on some metric: (CPU, HTTP load balancing, Stackdriver, etc)

* The number of instances within that group will then increase or decrease based on the metrics used.  
* The instance group are based on a template - all instances use that template and are therefore identical
    * This is great for stateless applications
* e.g. scales based on CPU usage hitting 60%, falling below again and extra instance will be shut after 10min

To create a managed instance group - you need a template which can be created via the google console or shell
* The instances you create need to be able to run your software so you need a startup script or something similar
    * Pre-build VM image
    * Dynamically install it at startup
    * Hybrid approach - prebuild some, and update on startup

Managed instance also have autohealing:
* VMs are stopped and recreated if found to be unresponsive based on different protocols
    * important to adjust healthcheck delay, you dont want it to check when your instances are still booting up or initialising your bootstrap code

## Usages

Can be used to perform a rolling update






