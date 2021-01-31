# Planning and Estimating with the Pricing Calculator

[[TOC]]

# Estimating Price of the Application
 https://cloud.google.com/products/calculator/

 The calculator can determine a rough monthly cost for out application, can fill in details:
 ## Compute Engine
 * Number of instances
    * OS
    * VM Class
    * Instance type
    * Location
* Disks
    * Space required
## App Engine
* Instances per Hour
* APIs required
## Kubernetes Engine
* Number of nodes
* Instance type
* Hours per day running, days per week etc
## Cloud Storage
* Location
* Storage required
* Nearline Storage
## Networking
## BigQuery
* On demand or flat rate
* Data read per month by query

etc
The cloud calculator will go through every single service available from GCP and will estimate 

From the estimate of the app - Spanner and BigQuery take the most money, spanner with $2400 a month to run