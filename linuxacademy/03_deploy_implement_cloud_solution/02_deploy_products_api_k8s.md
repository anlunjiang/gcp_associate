## Deploying the products API on Kubernetes

[[TOC]]

Products API is used to fetch all the product images the users uploads - which runs on K8s, reading data from bq and caching that data in cloud storage as a form of caching

[products/cloud/setup.sh](../../content-google-cloud-engineer-master/products/cloud/setup.sh)

Most of the script in the beginning is to create a service account - making sure it only has the privileges that the accounts absolutely needs: This is the `principle of least privilege`  
* Note that compute engine instances can also use a specific service account with all the relevant permissions 
* Google tools and libs have a set of rules used  to locate creds, called `Application Default Credentials (ADC)`
    * First look for an environment variable called `$google_application_credentials` that points to the service account key 

### Usage

The code needs to be able to run queries against bq as well as manage objects in cloud storage:

```bash
# Create the service account - which will also create an email address (member id)
gcloud iam service-accounts create \
    $SERVICE_ACCOUNT_NAME \
    --display-name $SERVICE_ACCOUNT_NAME

# Setup the necessary permissions for the service account
gcloud projects add-iam-policy-binding $PROJECT_NAME \
    --role roles/bigtable.user \
    --member serviceAccount:$SA_EMAIL
# To use service account in containerised app we need to create a key to allow the code to login as service account
gcloud iam service-accounts keys create $SERVICE_ACCOUNT_DEST --iam-account $SA_EMAIL
# make sure your keys are not uploaded to github by putting it in the git ignore, this wil be generated in your local secrets folder
```

## Creating the K8s cluster
```bash
gcloud beta container clusters create $PRODUCT_CLUSTER_NAME \
    --project $PROJECT_NAME \
    --zone $PROJECT_ZONE \
    --no-enable-basic-auth \ # LEGACY FEATURE
    --machine-type "n1-standard-1" \
    --image-type "COS" \ # google provided image0
    --disk-type "pd-standard" \
    --disk-size "100" \ #GB
    --num-nodes "3" \
    --enable-cloud-logging \
    --enable-cloud-monitoring \
    --network $SERVICES_NETWORK \
    --subnetwork $PRODUCT_SUBNET \
    --addons HorizontalPodAutoscaling,HttpLoadBalancing,KubernetesDashboard \
    --enable-autoupgrade \ # ensure nodes always running latest version of K8s
    --enable-autorepair \
    --service-account $SA_EMAIL

# sets auth for google cloud auth registry - K8s doesnt work with dockerfiles, only with actual images
    gcloud auth configure-docker -q

# list all cluster running 
`gcloud beta container clusters list`
```
The key created for the service account is then shown in the ui as well - can change also if accidently compromised
This service account can be used by the  can now interact with bq and cloud storage 

## Deploy Products API

[products/deploy/deploy.sh](../../content-google-cloud-engineer-master/products/deploy/deploy.sh)

first runs the build/build.sh script: which creates a docker image from the dockerfile, just an alpine image with some sql code   
`docker build -t la-ace-products:0.1 -f "$(dirname "$0")/Dockerfile" "$(dirname "$0")/../"`

The docker image is then saved into the container registry for K8s to see it   
`docker tag la-ace-products:0.1 "gcr.io/$PROJECT_NAME/products"`  
`docker push "gcr.io/$PROJECT_NAME/products"`  
Tag for the google cloud repo and push

* Authenticate the Kubectl command - then we can start using it  
`gcloud container clusters get-credentials $PRODUCT_CLUSTER_NAME --zone $PROJECT_ZONE --project $PROJECT_NAME`

* Create a Kubernetse secret that stores the service account key
```bash
# This method does kubectl create secret and kubectl update secret all in one line 
kubectl create secret generic service-account-file \
    --from-file=../app/secrets/service_account.json \
    --dry-run -o yaml | kubectl apply -f -
```

* Deploy workload - which containers to run
    * in a yaml file - to specify the resources needed
        * each resources might have its own specifications

[products/deploy/workload.yaml](../../content-google-cloud-engineer-master/products/deploy/workload.yaml)

[products/deploy/service.yaml](../../content-google-cloud-engineer-master/products/deploy/service.yaml)


