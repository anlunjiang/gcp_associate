# Setting up cloud projects and accounts

[[TOC]]



TODO List:
* ~~Create Accounts~~
* ~~Create Project~~
* ~~Create IAM User for John with ML Developer Role~~
* ~~Enable ML API~~
* ~~Setup Stackdriver Account~~
* ~~Link GSuite identities to users~~
* ~~Create a Billing account~~
* ~~Setup Budget and Alerts~~
* ~~Install Cloud SDK~~
* Create Cloud Presentation

# Projects

Projects serve as a box (isolated environment) that will provide all the resources that we need - resources belong to a project
* You can setup different projects for different environemnts (dev, test, prod)
* Can also setup different billing and permissions

Each project has 3 identifiers:
1. Project name: Human friendly id - easy to recognise
2. Project number: Created by google as back-end ID
3. Project ID: Globally Unique, used by engineers, cloud sdk etc, we can set ourselves and cant be changed once set


## Hierarchy 

![External and Internal Load Balancers](../../images/cloud-folders-hierarchy.PNG =600x)

* Organisations: allows companies to have control over all projects that they own and managing permissions
    * Automatically created if using G Suite or Cloud identities
* Folders: allows projects to have hierarchy - good for mirroring company's org structure (Note that this is flat in reality)
* Projects: as described above - have resource assiciated with it

## Should I use an organisation?

Projects created without an organisation:
* Any projects created will be owned by the employee account, and will be deleted along with the account (if they leave the company)

* It is usually recommended to use/create an organisation!

## Deleting project
Either going to project settings or resource management under IAM & Admin
* It will then be scheduled for deletion within 30 days

# Creating Users and Assigning Roles

Adding users can be done in the IAM & admin page on the side of the google console page
Note:
* IAM does NOT serve as an identity provider 
    * It only allows us to control who can do what with what resource
    * Accounts must be known to google (gmail, googlegroups, gserviceaccounts, gsuiteUsers)

* If users aren't recognised and you're using gsuite, you can add them as a user to the gsuite account
* Once added you can set their role (Can add multiple)

## GCP Roles

There are 3 main types of roles in GCP
* Primitive Roles - These predate Cloud IAM, project only, and very generic (NOT RECOMMENDED)
    * Project Owner
    * Project Editor
    * Project Viewer
* Pre-defined Roles - Granular service-specific roles
    * e.g. ML Engine Developer
    * Very finite permissions
* Custom Roles - Like predefined but can be merged and very targeted - following **Principle of least privilege**
    * Users should only get the access they need and nothing more 

# Enabling APIs

Users would need to access some apis to test out and play around it
* You can see all the apis that are available for a project in the APIs & Services panel on the side
* You can then click an existing api to look at its traffic and methods
    * Good to diagnose api methods that take longer than usual / fail/success etc
* There is also a quota tab to look at how close the project is to hitting its quota limits

## API explorer
You can search for new apis and try them out in the api explorer
* Will give you all the methods and you can try it out by clicking on them 

# Stack Driver
A built in solution for monitoring, logging and debugging for your host project - has an entire stackdriver section on the console
* Uptime monitoring
* Performance metrics
* Alerting - email, sms -identify issues asap based on conditions
* AWS Monitoring

Stackdriver started as a 3rd part service in 2012 till Google bought it in 2014 and in 2016 integrated it in GCP
- This is the reason why stackdriver may jump across some tabs
- This is why you need a stackdriver account too and can associate your projects with it to monitor 

## Host Projects Best Practices

Stackdriver config data and other info is stored under host project. 
In case stackdriver is used for monitoring only 1 project, we can re-use the same as host project.
In the case it is being used to monitor more than 1 project it is advised to use a blank project as host proj.

## Stackdriver APM (Application Performance Management)
* Debug - debug live apps without impacting users
* Trace - Used to find performance bottlenecks - app/compute engine, k8s
* Logging - logging service for all our apps
* Error reporting - pulls all errors from logging and aggregates them into one place
Stackdriver logging and monitoring agents on each of your VM instances - collects more data from your VM instances icnluding metrics and logs from 3rd party apps 

## Groups
Stackdriver groups allow us to monitor multiple resources as if they are the one entity - great for applications

# Summary
* Resources - Granular Building Blocks for services and APIs
    * Compute Engine Instance
    * Cloud Storage
    * App Engine Instance
* Together: Resources are services - services and apis make up projects
* Projects - isolated box that stores resources 
    * Everything happens in the context of a project
    * Project structure can vary, but you can one project per application or role
        * Or can reflect internal structure of the organisation
* IAM Identity Access & Management
    * Who can do what with what resource
    * Roles:
        * Primitive - Project level - outdated
            * Owner
            * Editor
            * Viewer
        * Predefined - Resource level
            * BigQuery Admin
            * BigQuery Data Editor
            * BigQuery Data Viewer etc
        * Custom - mix and match predefined roles and edit
* Managing APIs
    * Can be done via console or via SDK
* Stackdriver - Can monitor multiple projects
    * Monitoring
    * Logging
    * Debugging
    * Error Aggregation
    * Needs a host project to handle metadata - best practice is to use a dedicated stackdriver project for that if you plan to monitor multiple projects