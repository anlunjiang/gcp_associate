# Deploying the Ads API on Compute Engine with Deployment Manager 

[[TOC]]

The ads api is an application written in GO that pulles ads from the spanner database and returns them as JSON - to be rendered on the frontend. The code listens on port 8002

* Compile the code and deployed into a compute engine managed instance group, sitting behind a load balancer

[ads/cloud/setup.sh](../../content-google-cloud-engineer-master/ads/cloud/setup.sh)

* This sets up a spanner database 
    * Creates a new spanner instance - a container for our databases
        * `gcloud spanner instances create $PRODUCT_DB_INSTANCE_NAME`
        * need to choose if regional or multi regional (google will handle replication and more availability, but more costly)
    * Create a new database within that instance  
        * `gcloud spanner databases create $PRODUCT_DB_NAME --instance=$PRODUCT_DB_INSTANCE_NAME`
    * Create the table 
        * `gcloud spanner databases ddl update $PRODUCT_DB_NAME --instance=$PRODUCT_DB_INSTANCE_NAME --ddl=<SQL TO RUN>`
        * Using the DDL subcommand (Data Definition Language)
    
# Deployment Manager

Deployment manager is Google's Infrastructure as Code service - making API calles for you
* Using declarative Syntax - services you want and the settings for them
* Is a way to easily reproduce projects - easy to use the same code moving from dev to prod 
* Create templates (yaml) that define the different resources used (Compute Engine Resource)
    * This template is then given to the deployment manager which figures out the details and creates the resources 

 Deployment manager will:
 * Create a Managed Instance Group
 * Create an auto scaler
 * Create a Load Balancer

 [ads/cloud/deploy.sh](../../content-google-cloud-engineer-master/ads/cloud/deploy.sh)
 * copies the local ad image to the private bucket and adds a row to the spanner table (keyvalue pairs to insert columns)
    ```bash
    gcloud beta spanner rows insert \
    --instance=$PRODUCT_DB_INSTANCE_NAME \
    --database=$PRODUCT_DB_NAME \
    --table=ads \
    --data=AdID='first-ad',Company='Linux Academy',FileName='juice.jpg',Name='Juice Ad',PromisedViews=1000,TimesViewed=0,Timestamp='spanner.commit_timestamp()'
    ```
`gcloud deployment-manager deployments create ad-service-deployment --config ads.yaml`  
This creates all the compute engine infrastructure - you cant then use the update command to change things later 
* This is based on the config ads.yaml which points to 3 jinja templates - `see templating engines`
* The order of the metadata doesnt matter to the deployment manager  

## Jinja Templates

> ### instance.jinja
* Creates an Instance template:
    * The bootstrap code copies the image uploaded to the bucket and saves to app dir 
    * Can define the machine type, disks and properties, network interface, which subnets
    * setup the service account instance will use and its permissions 

> ### autoscaler.jinja

* Creates a managed instance group and an autoscaler
    * set zone and target size of the group 
    * Specify the fully qualified instanceTemplate resource link for the instance template created in the instance.jinja file 
    * Create a named port so load balancer knows which port app is listening on 
    * Create an autoscaler resource - specify the target (the instance group to scale)
        * set max number of instances based on cpu usage 

> ### loadbalancer.jinja

* Creates a HTTP Load balancer service to distribute the traffic from the frontend 
    * specify backend service which tells it which instance group we want to distribute traffic to 
        * same port name we used in instance group resource 
    * How it distributes traffic - CPU utilisation? 
    * setup healthcheck 
    * setup forwarding rules that map port 80 to 8002 on the instances
        * HTTP Proxy which requires a URL map which needs to know which backend we are mapping to  
    * Make sure traffic can flow between the load balancer and the instances setting up a FIREWALL RULE TCP
        * source ranges being googles load balancer

```
              User
                |
        Forwarding Rule Port 80
                |
            HTTP Proxy
                |
            URL MAP
                |
        Backend Service 
    Named port: Application-port 8002
```

## Bringing it all together

> ### ads.yaml
 [ads/cloud/ads.yaml](../../content-google-cloud-engineer-master/ads/cloud/ads.yaml)
 * Imports the 3 jinja templates and uses them as a resource type before setting the properties 
 * It sets up the properties that we look up inside the jinja templates e..g for instance.jinja it looks up   
    `gs://{{ properties["privateBucket"] }}/{{ properties["adBinName"] }}` for the image in the private bucket to copy to apps directory
    * ads.yaml will specify theses properties for that template:
    ```yaml
    properties:
        region: us-central1
        zone: us-central1-b
        prefix: ads-service
        privateBucket: fs2-private-bucket
        publicBucket: fs2-public-bucket
        spannerDatabase: fs2-app-spanner-db
        spannerInstance: fs2-app-spanner-instance
        network: fs2-app-network
        subnet: fs2-ad-app-network-subnet
        projectID: ace-demo-2
        adBinName: app
        serviceAccount: 53612557816-compute@developer.gserviceaccount.com
    ```

`gcloud deployment-manager deployments list` list all deployments  
`gcloud deployment-manager deployments describe <name of deployment>` shows all the metadata for that deployment  
`gcloud compute forwarding-rules list --filter='name:"<>nameofrule"' --format='value(IPAddress)'` to see the load balancer external ip address

# Summary
* Created a spanner instance as a prereq to deploy the application 
* Used deployment manager to create all the compute engine resources we need in one line 
    * Managed Instance Group
    * AutoScaler
    * Load balancer service
* Used startup script metadata to download code from cloud storage and set it up as a service in the instances
    * instances use the startup code to download the code on startup and make sure service is created

# Templating Engines 
Performs a find and replace. We have a text file we want jinja to go through and replace the variables with the values set.   
The text is just a yaml file with jinja expression syntax 
* Benefit is that we are able to run the same template with different values rather than hardcoding everything 
* For the instance.jinja we can bootstrap our code to run code on startup, in metadata with key: startup-script
    * This is the same as creating instance templates in the UI but just programmatically 

![create_instance_template](../../images/create_instance_template.PNG =900x)

