# Google Kubernetes Engine Hands on Lab

[[TOC]]

You’ve been tasked with creating a Kubernetes Engine cluster with four nodes in the us-central1-a zone. To test the cluster, you’ll need to containerize an app and take all the steps necessary to deploy and expose it on the cluster.
Read Bottom up:

![module_hierarchy](../../images/gke01_lab_diagram.png =400x)

#  Enable Kubernetes Engine API

Click on APIs and Services and Library to enable the Kubernetes Engine API

# Clone the repo to retrieve app files

`git clone https://github.com/linuxacademy/content-gc-essentials`  
`cd content-gc-essentials/gke-lab-01`

# Create the Docker image

```bash
##############################################################################################################

### dockerfile ###

cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ cat dockerfile
# The Dockerfile defines the image's environment
# Import Python runtime and set up working directory
FROM python:3.7-slim
WORKDIR /app
ADD . /app

# Install any necessary dependencies
RUN pip install -r requirements.txt

# Open port 80 for serving the webpage
EXPOSE 80

# Run app.py when the container launches
CMD ["python", "app.py"]

##############################################################################################################

### app.py ###

cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ cat app.py
# The Docker image contains the following code
from flask import Flask
import os
import socket
app = Flask(__name__)
@app.route("/")
def showPinehead():
    html = "<div style='text-align:center;margin:20px;'><h1>Greetings from Linux Academy!</h1><img src='https://storage.googleapis.com/la-gcp-labs-resources/essentials/Logo-Pinehead-NVY.png' width='40%' alt='Pinehead @ Linux Academy'></div>"
    return html
if __name__ == "__main__":
  app.run(host='0.0.0.0', port=80)

##############################################################################################################

### Building Docker image ###

cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ docker build -t la-container-image .
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM python:3.7-slim
3.7-slim: Pulling from library/python
68ced04f60ab: Pull complete
d1d516f83cca: Pull complete
e21330ef4680: Pull complete
d073c4206257: Pull complete
73e32695886c: Pull complete
Digest: sha256:a4938310ae4b76ff20a27e225095f9cb574fc88fe820893d12aa5b3c6405b1e9
Status: Downloaded newer image for python:3.7-slim
 ---> 84de2ffd919d
Step 2/6 : WORKDIR /app
 ---> Running in bd11e9cd92f7
Removing intermediate container bd11e9cd92f7
 ---> 4441ea67b88c
Step 3/6 : ADD . /app
 ---> b7b426f84719
Step 4/6 : RUN pip install -r requirements.txt
 ---> Running in 7f3360edf212
Collecting flask>=0.12.3
  Downloading Flask-1.1.1-py2.py3-none-any.whl (94 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.0-py2.py3-none-any.whl (298 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
Collecting click>=5.1
  Downloading click-7.1.1-py2.py3-none-any.whl (82 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp37-cp37m-manylinux1_x86_64.whl (27 kB)
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, click, itsdangerous, flask
Successfully installed Jinja2-2.11.1 MarkupSafe-1.1.1 Werkzeug-1.0.0 click-7.1.1 flask-1.1.1 itsdangerous-1.1.0
Removing intermediate container 7f3360edf212
 ---> dcb5f3ebef59
Step 5/6 : EXPOSE 80
 ---> Running in 91323829270d
Removing intermediate container 91323829270d
 ---> 619309929e4d
Step 6/6 : CMD ["python", "app.py"]
 ---> Running in 994ec6efbc14
Removing intermediate container 994ec6efbc14
 ---> e39fbe29aeff
Successfully built e39fbe29aeff
Successfully tagged la-container-image:latest
cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ cat dockerfile
# The Dockerfile defines the image's environment
# Import Python runtime and set up working directory
FROM python:3.7-slim
WORKDIR /app
ADD . /app
# Install any necessary dependencies
RUN pip install -r requirements.txt
# Open port 80 for serving the webpage
EXPOSE 80
# Run app.py when the container launches
CMD ["python", "app.py"]
```

Now you have run the docker file, you have an image:

```bash
cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ docker image list
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
la-container-image   latest              e39fbe29aeff        8 minutes ago       188MB
python               3.7-slim            84de2ffd919d        12 days ago         179MB

#####################################

cloud_user_p_505aff@cloudshell:~/dev/content-gc-essentials/gke-lab-01 (creating-and-61-4b8db1)$ gcloud auth configure-docker
WARNING: Your config file at [/home/cloud_user_p_505aff/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud"
  }
}
Adding credentials for all GCR repositories.

docker tag la-container-image gcr.io/<PROJECT_ID>/la-container-image:v1
docker push gcr.io/<PROJECT_ID>/la-container-image:v1

```
#  Create the Kubernetes Engine cluster
#### In the Cloud Shell, enter the following commands:
`gcloud config set compute/zone us-central1-a`  
`gcloud container clusters create la-gke-1 --num-nodes=4`  

NAME | LOCATION | MASTER_VERSION | MASTER_IP | MACHINE_TYPE | NODE_VERSION | NUM_NODES | STATUS
-----|----------|----------------|-----------|--------------|--------------|-----------|-------
la-gke-1 | us-central1-a | 1.14.10-gke.17 | 35.184.136.33 | n1-standard-1 | 1.14.10-gke.17 | 4 | RUNNING

#### Enter the following commands to deploy the app, substituting your project ID where indicated:  
`gcloud container clusters get-credentials la-gke-1`  Now you can use kubectl cmds to deploy  
`kubectl run la-greetings --image=gcr.io/<PROJECT_ID>/la-container-image:v1 --port=80`  

#  Expose the deployed workload

#### In the Cloud Shell, enter the following commands:  
`kubectl expose deployment la-greetings --type=LoadBalancer --name=la-greetings-service --port=80 --target-port=80`  
`kubectl get services la-greetings-service`  
#### In a browser tab, visit the external IP address listed
NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) | AGE
-----|------|------------|-------------|---------|----
la-greetings-service | LoadBalancer | 10.59.249.251 | 34.67.192.140 | 80:32626/TCP | 84s


![module_hierarchy](../../images/k8s_hosted.png =400x)





