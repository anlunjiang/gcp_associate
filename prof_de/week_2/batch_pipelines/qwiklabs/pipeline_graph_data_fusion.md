# Building and Executing a Pipeline Graph with Data Fusion

Enable the cloud data fusion api: `gcloud services enable datafusion.googleapis.com`

* Cloud Data Fusion translates your visually built pipeline into an Apache Spark or MapReduce program
    * Executes transformations on an ephemeral Cloud Dataproc cluster in parallel

![df](../images/datafusion.PNG =1000x)

* Graph is almost like alteryx - click and drag
    * Use joiners to join data together 
* Once pipeline has been made you can deploy and run
    * Dataproc cluster will be spun up to run your pipeline - note you can choose batch or real time running

Note that you can create tables from queries directly in BQ - using the query formatter 

