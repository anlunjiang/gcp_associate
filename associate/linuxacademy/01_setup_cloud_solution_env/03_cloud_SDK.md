# Cloud SDK (Software Development Kit)

[[TOC]]

# Installing the Cloud SDK
The Cloud SDK is a set of command line tools to interact with services - used to automate and manage GCP services  
You install this locally or on a docker container
This allows the gcloud language options to be available for your local development

`gcloud init` to start the process and login to activate the sdk  
This will provide a link and code and you connect the SDK to a GCP accountv - before asking you to choose a default project

* To find out what project is your default:
> `gcloud config list` - which will show the account and project 
> `gcloud info` - gives more information on the SDK and configuration

# Using the Cloud SDK
The SDK is based on the multiple components - glcoud is a component

```
                   Cloud SDK
                        |
                   Components
                        |
    Core ------------GCLOUD --------- BQ ----------------- GSUTIL
    |                   |               |                      |
Shared Library        Alpha          BigQuery Component      Interact with cloud storage
between components     Beta
```

 `gcloud components list` - List componenets ready to be used  

 ## GCLOUD syntax
 gcloud syntax is very modular for ease of use, it consists of gcloud, a group and a command
> `<component><group><command>`  
> `gcloud auth list`  
A group is a collection of commands usally grouped by service, name or responsibility 

### Syntax help
`gcloud config--help`  
`man gcloud_config`  
`man -k gcloud` - prints out all pages that contain gcloud

# Properties and Configurations
* Properties: configure the behaviour of gcloud
    * account
    * projects
* Configurations: named set of properties
    * Can easily switch between a set of properties 
    * > `gcloud config configurations list` onlyt one by default named "default"

e.g.

`gcloud config set compute/zone us-east-1`  
To set the default zone of a project for the default config 
