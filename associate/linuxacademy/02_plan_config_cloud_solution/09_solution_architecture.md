# Linux Academy Development Problem and initial approach

# High Level
* User goes to the website and the homepage will be user uploaded products that users are looking to buy or sell
* They can use a submission form to submit image of a product they want to buy or sell
* On the side panel there will be a featured advertisement

# Solution

![soln](../../images/solution.PNG =1000x)

* Front end UI is run on App Engine
* When a user makes a submission the app running in App Engine will take the form values (name, price, image etc) and publish it as a Pub/Sub topic 
* Cloud function gets called everyimte there's a new message and (NodeJs6/8, Python3)
    * It converts the message from b64 string back into a JSON object, image is still b64 str
* Cloud vision api receives image and will guesses what the image is with other metadata about the image 
* Saves data into BigTable (Best guess as RowKey)
    * THIS IS OVERKILL - too expensive for such little data (BigTable works with PetaBytes)
    * Cloud Datastore would be better
* BigTable is paired with BigQuery (used in this case as a query engine)
* Products API: in a docker container on a K8s cluster
    * Doesnt make sense to run a bigquery query everytime a user refreshes, so it gets cached in cloud storage and refreshed every 5 minutes
* Ads API: Run in a VM Instance - using spanner for scalability as a database
    * OVERENGINEERED - spanner is expensive, no ads yet, this just adds cost for running the code 

* Plan is to have the two apis in their own subnets that can interact with each other 

