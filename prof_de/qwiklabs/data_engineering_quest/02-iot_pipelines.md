# Building an IoT Analytics Pipeline on Google Cloud

* Cloud IoT Core is a fully managed service that allows you to easily and securely connect, manage, and ingest data from millions of globally dispersed devices

Cloud IoT Core has two main components:

* A device manager for registering devices with the service, so you can then monitor and configure them.
* A protocol bridge that supports MQTT, which devices can use to connect to Google Cloud.

* Cloud Pub/Sub is an asynchronous global messaging service. By decoupling senders and receivers, it allows for secure and highly available communication between independently written applications. Cloud Pub/Sub delivers low-latency, durable messaging.

* After creating a PS topic - you can create a dataflow job to write from that topic to say bigquery - there are template dataflow jobs you can use that will do this for you

> `curl https://sdk.cloud.google.com | bash` - install the latest version of the Cloud SDK 

## Create a registry for IoT devices

* To register devices, you must create a registry for the devices. The registry is a point of control for devices.

> `gcloud beta iot registries create iotlab-registry \`  
> `   --project=$PROJECT_ID \`  
> `   --region=$MY_REGION \`  
> `   --event-notification-config=topic=projects/$PROJECT_ID/topics/iotlab`  

# Create a Cryptographic Keypair
To allow IoT devices to connect securely to Cloud IoT Core, you must create a cryptographic keypair.

* In your SSH session on the iot-device-simulator VM instance, enter these commands to create the keypair in the appropriate directory:

> `cd $HOME/training-data-analyst/quests/iotlab/`  
> `openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem \`  
> `    -nodes -out rsa_cert.pem -subj "/CN=unused"`  

# Add simulated devices to the registry
* For a device to be able to connect to Cloud IoT Core, it must first be added to the registry.

> `gcloud beta iot devices create temp-sensor-buenos-aires \`  
> `  --project=$PROJECT_ID \`  
> `  --region=$MY_REGION \`  
> `  --registry=iotlab-registry \`  
> `  --public-key path=rsa_cert.pem,type=rs256`  

* Enter this command to create a device called temp-sensor-istanbul:  

> `gcloud beta iot devices create temp-sensor-istanbul \`  
> `  --project=$PROJECT_ID \`  
> `  --region=$MY_REGION \`  
> `  --registry=iotlab-registry \`  
> `  --public-key path=rsa_cert.pem,type=rs256`  

# Running devices

* Telemetry data will flow from the simulated devices through Cloud IoT Core to your Cloud Pub/Sub topic. In turn, your Dataflow job will read messages from your Pub/Sub topic and write their contents to your BigQuery table.

* Cloud IoT Core supports two protocols for device connection and communication:
    * HTTP
    * MQTT (Message Queue Telemetry Transport)

