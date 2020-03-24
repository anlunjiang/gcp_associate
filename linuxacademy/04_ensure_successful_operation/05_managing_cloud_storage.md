# Working with cloud storage

[[TOC]]

Cloud storage is a verstaile solution for many purposes:
* Cheap backup solution
* Store archives for compliance reasons
* Serve assets for web applications
* Serve static websites

`gsutil ls` - list all the buckets for the default project   
`gsutil mb -c regional -l us-east1 gs:/<bucket_name>/` - make a bucket that is regional storage class  

For this case it is used to store ML models

In the console - you can change the default storage class for objects that we dont specify a default storage class
* You cannot switch between regional and multi regional though - but can switch to nearline/coldline
    * Note this doesnt change existing objects - only new uploaded objects with no default class

### Cloud storage allows you to implement a lifecycle rules

Lifecycle rules allow us to switch files older that certain number of days old to use nearline storage instead 
* It allows us to take action after a condition is met
    * Age - in days, TTL time to live
    * Created before - created before midnight of a date
    * isLive - matches only live objects if true vv if false
    * MatchesStorageClass - object in bucket is stored as the specified storage class
    * NumberOfNewerVersions - relevant to versioned objects - Satisfied if an object has at least N versions newer than it
* and take action
    * Delete
    * SetStorageClass - change the storage class (regional to nearline/coldline for example)

To configure in CLI we need a JSON file that contains the rules 

```JSON 
# lifecycle.json
{
    "lifecycle": {
        "rule": [
            {
                "action": {
                    "type": "SetStorageClass",
                    "storageClass": "NEARLINE"
                },
                "condition": {
                    "age": 2,
                    "matchesStorageClass": [
                        "REGIONAL"
                    ]
                }
            }
        ]
    }
}
```

`gsutil lifecycle set lifecycle.json gs://<bucket_name>` - set lifecycle rules to objects in bucket  
Now every object in that bucket with a storage class of regional that is over 2 days old will be set to nearline storage class  

# Upload and move a file

`gsutil cp test.txt gs://<bucket_name>` - copy a text file to a bucket in cloud storage  
`gsutil mv gs://<bucket_name>/test.txt gs://<bucket2>/` - move a file from one bucket to another one

Note that by default: Everything (buckets and objects are private)

## Changing object permissions

You can change objects to public - by changing permissions by setting the user to the `AllUsers` token and view access

![cloud_storage_permissions](../../images/cloud_storage_permissions.PNG =800x)

And it will provide a public link to that asset. You can easily make it private again by removing that user with allUsers name - but this could take a minute or two.

## Make data available temporarily

You can use signed URLs to achieve this, instead of making a bucket public
* To create a signed URL you need a service account key (create a new service account in IAM and create a JSON key)
* `gsutil signurl -d 3m key.json gs://<bucket_name>/test.txt` - create a signed url that expires in 3 minutes using the key.json from the service account for the object test.txt
* This will then generate a signed URL for anybody in the world that is only valid for 3 minutes only

# Summary
* Make bucket using `gsutil mb`
* By default all buckets and objects are private 
* Buckets can have lifecycles
    * lifecycles consist of one or more conditions and actions 
* Use allUsers group with Reader permissions to make somthing public 
* Use signed URLs to share private data temporarily