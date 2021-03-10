# Creating a Data Transformation Pipeline with Cloud Dataprep

* Cloud Dataprep by Trifacta is an intelligent data service for visually exploring, cleaning, and preparing structured and unstructured data for analysis

* Dataprep can select different sources of data including big query tables and load them into the pipeline. You can then edit the "recipe" for it and explore the data
    * When editing the recipe you can explore the histograms of data - which will show you the most common values of the column

![dataprep](../images/dataprep.PNG =600x)

* Statistics on each column can be found by clicking a column and on column details
    * This includes the 
* Columns can have grey bars which shows the proportion of missing data
* Red bars mean mismatched values
    * While sampling data, Cloud Dataprep attempts to automatically identify the type of each column
    * If you do see a red bar, then this means that Cloud Dataprep found enough number values in its sampling to determine the type but not all values are of that type

* Everything you do will be recorded in the recipe as a step this includes
    * Filters, where statements, deletions, dedupes etc
* Once a recipe has been created you can schedule it
    * Scheduled jobs can have scheduled destinations that are separate from manual destinations
        * You can have dataprep jobs save to GCS for a one time run but the schedule can write to bq (can append, truncate and write etc)

* Note: the dataprep is actually a dataflow pipeline - running on Apache beam
    * Whilst running you can see the dataflow job live and the dags that it has created:

![dataprep](../images/dataprep_to_dataflow.PNG =800x)

* You can see once the job is finished the resulting bq table it has created:

![dataprep](../images/dataprep_result.PNG =800x)

* Note the added column and data transformations etc! 

