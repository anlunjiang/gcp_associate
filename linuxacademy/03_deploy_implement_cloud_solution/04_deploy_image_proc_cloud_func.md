# Deploying the image processor with cloud functions

This is the code that will be called whenever a message is published to the pub sub topic we setup earlier 

JavaScript code will running in the Cloud function using the node.js 6 runtime

* When a message is published to the pub sub topic, it will run our code
    * deserialising the user uploaded image from the message and sending it to cloud vision for logo, web and labelled detection 
    * This cloud vision data is returned and stored with the original message in bigtable
    * This bigtable data is what is queried by BQ on the products api

 [image_parser/app/index.js](../../content-google-cloud-engineer-master/image_parser/app/index.js)

 * This is not using the service account we created, but the default one for cloud functions instead.

 [image_parser/deploy/deploy.sh](../../content-google-cloud-engineer-master/image_parser/deploy/deploy.sh)

 ```bash
 gcloud beta functions deploy $FUNCTION_NAME \
    --entry-point=imageParser \ # name of the actual function in our code
    --source=$SOURCE_LOCAL_FOLDER \
    --stage-bucket=$PRIVATE_ASSETS \ # where to upload the code to
    --runtime nodejs8 \
    --trigger-resource=$PUB_SUB_TOPIC \
    --trigger-event="google.pubsub.topic.publish" \ # code executes everytime pub sub is triggered
    --project=$PROJECT_NAME \
    --region=$PROJECT_REGION \
    --set-env-vars=BIGTABLE_INSTANCE_ID=$BIGTABLE_INSTANCE_ID,BIGTABLE_TABLE_ID=$BIGTABLE_TABLE_ID,CLOUD_STORAGE_BUCKET=$PUBLIC_ASSETS
 ```

 * Cloud Functions can be associated witha specific trigger. The trigger type determines how and when your function executes. Cloud Functions supports a range of triggers:
    * HTTP
    * Cloud Pub/Sub 
    * Cloud Storage
    * Direct Triggers
    * and more

`gcloud functions list` list all the current cloud functions