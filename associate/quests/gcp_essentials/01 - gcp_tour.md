# Tour of GCP Platform

## Google Cloud platform console
Central dev hub of most projects - includes web console and status dashboard. Uses include:
* Enabling APIs for a projects
* Creating and deleting buckets
* Uploading, Downloading and Deleting Projects
* Managing IAM Policies

 Access the Console by either clicking on the shell icon within the dashboard or going to: https://console.cloud.google.com/

 Consoles with Bucket only access can be assigned to non project members - able to work only with objects within that bucket: https://console.cloud.google.com/storage/browser/[BUCKET_NAME]

 Like with Buckets, Object only console links exist - which can differ in format depending on the permissions of that object. Public links look like:
 https://console.cloud.google.com/storage/browser/_details/[BUCKET_NAME]/[OBJECT_NAME]

 ## Storage

 **GCP uses a flat namespace to store data**: However cloud console can mimic a folder hierarchy for familiarity
 object naming follows hierarchy, but not actual hierarchy is implemented

 ## Projects
 An organising entity: containing resources and services; pool of VMs, databases and a network. GCP project_id's are globally unique
 > Resources:  
 Physical assets, computers, hard drives, VMs, contained in data centers
 Each data center location is in a **Region** which contains **Zones**
 *  For example, zone a in the East Asia region is named asia-east1-a.
 * Distribution of resources help with redundancy for failure and latency for customers

**Global Resources** - Can be accessed globally - preconfigured disk images, snapshots
**Regional Resources** - Accessed only in the same Region, include static external IP addresses 
**Zonal Resources** - VM Instances, types and disks

> Services: Hardware and Software products in GCP are now services that are provided
* **Compute**: houses a variety of machine types that support any type of workload. The different computing options let you decide how you want to be with operational details and infrastructure amongst other things.
* **Storage**: data storage and database options for structured or unstructured, relational or non relational data.
* **Networking**: services that balance application traffic and provision security rules amongst other things.
* **Stackdriver**: a suite of cross-cloud logging, monitoring, trace, and other service reliability tools.
* **Tools**: services for developers managing deployments and application build pipelines.
* **Big** Data: services that allow you to process and analyze large datasets.
* **Artificial Intelligence**: a suite of APIs that run specific artificial intelligence and machine learning tasks on the Google Cloud platform.

## Roles and Permissions

Cloud Identity and Access Management (IAM) service can be used to inspect and modify roles and permissions. This can be viewed easily on the dashboard  side panel **IAM & admin**  


| Primitive Role | Permissions |
|----------------|-------------|
| viewer         | Permissions to read-only actions that do not affect stat (viewing but not modifying existing resources or data) |
| editor         | All viewer permissions + permissions for actions that modify state, such as changing existing resources         |
| owner          | All editor permissions + able to manage roles and permissions for a project, setup billing for a project        | 

Note that these are all **primitive roles**. It is better to now use predefined or custom roles 

## APIs and Services

Can be called either directly or via client libraries. They shuld have info on  usages and documentation

## Cloud Shell

An in-browser command prompt execution environment that is used to manage resources and services within a GCP project

> `gcloud auth list`  

```Credentialed Accounts
ACTIVE  ACCOUNT
*       gcpstaging23396_student@qwiklabs.net
To set the active account, run:
    $ gcloud config set account `ACCOUNT` 
```

The main GCP toolkit is **gcloud** used for actions like resource management and authentication

