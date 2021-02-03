# Exploring a BigQuery Public Dataset

Storing and querying massive data can be expensive. 
Fully managed Data Warehouse using SQL like syntax
* Pay for what you actually use:
    * streaming updates (Billing exports)
    * Queries you run BASED ON DATA READ
* You can calculate cost of the query by calculating the bytes to be read
    * CLI --dryrun returns number of bytes read
    * REST API can set --dryrun parameter to be true
    * UI Query validator
    * GCP Price calculator can then be used to figure out the cost 

Big query has a query validator that shows you if you have any syntax issues - it also estimates how many MiB will be run
* You need to create a dataset (basically a database) your query syntax will be select * from dataset.table
    * Your schema when creating the table can be set to automatically detect which is handy
    * when uploading from file - you can select the type 
    