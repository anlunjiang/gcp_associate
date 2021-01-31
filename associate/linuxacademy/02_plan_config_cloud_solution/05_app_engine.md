# App Engine

[[TOC]]

# App Engine 

App engine runs applications that are intended to be interacted with over http or https on a highly scalable managed platform 
* Great for web applications and mobile backends
    * Application versioning
    * Traffic splitting
    * Caching
    * Can use cloud datastore as it can scale 

## Version
App engine started off as one version: now called the "Standard Environment" which can use different language runtimes (Python 2.7, Java 8, Go 1.9. PHP 5.5 etc)
* However you cannot customise the runtime, install 3rd party binaries, write to local disk or SSH
* But they are cheap and have really fast startup - easy to scale

There are "Flexible Environments" that provide managed runtimes and customisation with Dockerfile

# Application Structure 
![module_hierarchy](../../images/modules_hierarchy.png =800x)

When you enable app engine for a project, it will create an application
* One application per Project
* Applications can be broken down into services
* Services can have multiple versions - easy to deploy and rollback
* Standard runtime: Versions run on instances (predefined hardware)
* Flexible runtime: Versions run on containers on top of compute engine instances

Note that when setting region for your application you can only set it once - so set the closest/best

# Deployment

Once you have an application for your project:
* You can clone your code from a github repository and bring it to the VM running the shell
* app.yaml will configure your application
    * this will show the runtime and any handlers
    * can listen for all web requests and send to the code
* dev_appserver,py app.yaml to run on the local dev server
* to deploy in app engine: `gcloud app deploy`
    * This uploads the code from github to appengine






