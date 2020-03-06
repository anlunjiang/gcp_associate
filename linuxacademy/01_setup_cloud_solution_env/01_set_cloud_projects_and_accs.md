# Setting up cloud projects and accounts

[[TOC]]



TODO List:
* Create Accounts
* Create Project
* Create IAM User for John with ML Developer Role
* Enable ML API
* Setup Stackdriver Account
* Link GSuite identities to users
* Create a Billing account
* Setup Budget and Alerts
* Install Cloud SDK
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

