# Deploying the Front-end in App Engine 

[[TOC]]

The Frontend is an App Engine Standard Env written in Go: when a user submits an image they want to buy/sell - it serialises it to a JSON object and publishes to a pub/sub topic, which is listened to by cloud funciton

 [frontend/cloud/setup.sh](../../content-google-cloud-engineer-master/frontend/cloud/setup.sh)  
 `gcloud app create --project=$PROJECT_NAME --region=$APP_ENGINE_REGION -q` Create the app engine instance in the project
 * Only one instance is allowed per project
 * Cannot delete, but you can disable to stop charging
 * Cannot change the region - which is app engines own region naming scheme

## Deploying the code base

[frontend/cloud/deploy.sh](../../content-google-cloud-engineer-master/frontend/cloud/deploy.sh)  
`gcloud app deploy $GOPATH/src/github.com/linuxacademy/frontend/app.yaml`

App engine is a platform as a service, more control can be had with configuration files for configuring the application with app.yaml (xml for java) file 

[frontend/app/app.yaml](../../content-google-cloud-engineer-master/frontend/app/app.yaml)  
 This maps URLS as static directories,  this allows us to serve static files. It also sets the pubsub topic so it can publish messages to that topic

## So far 
Our app is broken into multiple services, with the frontend tying them together:
* JavaScript makes call to the products API and ads API and then render the results in the browser 
    * The JS code needs access to the APIs [frontend/app/public/assets/js/items.js](../../content-google-cloud-engineer-master/frontend/app/public/assets/js/items.js)  
    * The IP addresses in the items.js file need to reflect the correct ip addresses (products application running on K8s, LB ip)
        * `kubectl get svc -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'`* 
        * `gcloud compute forwarding-rules list --filter='name:"ads-service-forwarding-rules"' --format='value(IPAddress)'`
        This grabs the ip of the ads service behind the global load balancer 

## Once deployed

Once the app engine has been deployed, you can run `gcloud app browse` it will show your website

Once setup, you can then upload some images to your site and you should be able to pick up some logs  
`gcloud functions logs read` to read most recent logs returned  
`cbt --project find-seller-app-dev --instance find-seller-bt-name-dev read items` - this will show all the big table data separated by row key 

* Data is therefore flowing from our frontend into pub/sub
* its then being processed by cloud function which passes the image to cloud vision api
* saving the data into bigtable 

You can also use the UI to use bigquery to query the same data:  
[products/app/products.sql](../../content-google-cloud-engineer-master/products/app/products.sql)    

![bq](../../images/bq.PNG =900x)

This means that bq is able to query the table and return the results 

### Refreshing the page now will show the images being hosted