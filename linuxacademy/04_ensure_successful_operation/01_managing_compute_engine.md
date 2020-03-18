# Managing Compute Engine Instances 

[[TOC]]

# Creating an Instance

## Create a development server for the data team
* Create a new project and then linking the billing account for it 
`gcloud projects create --organization=<org_id> <name_of_project>`  
`gcloud organization list` to find list of your organisations  
`gcloud beta billing accounts list` list all the billing accounts  
`gcloud beta billing accounts projects link <project_id> --billing-account=<billing_acc_id>`  
link a billing account to a project 

* Create a new VPC network for this project 
    * set ip-address range for custom
    * Set private Google access to "on": Dont need a public IP to access GC services
    * Create a VM Instance in Compute Engine
    * Allow SSH connectivity by configuring firewall rules
        * Create an ingress rule (for incoming traffic)
        * Set targets: which instances this rule applies to, two ways to do this:
            * By Tag - by marking instances with a tag you can apply firewall rules on them!
            * By service account usage
        * Set Source Filter: Who can interact with the target instances, instances in/with the allowed:
            * IP ranges
            * Subnets
            * Source Tags
        * Set Port for 22 (tcp:22) SSH
    * Once done, any instances with the tag will have that rule applied to it

# SSH

## Connecting to compute instance over SSH on Cloud SDK
`gcloud compute instances list` find the instance you want to connect to  
`gcloud compute ssh <linuxuser>@<nameofserver>` ssh into a gcloud instance - uploads ssh key for us, you can also add `--dryrun` to see what it's doing under the hood. This produces a project wide SSH key for us which can be found under metadata in the console 
* There is also support for 3rd party SSH software too if you make an SSH key `ssh-keygen -t rsa test-keys -C Anlun`
* We can then add the public key to the SSH instance metadata via a ssh_users.txt:
    * `[USERNAME_1]:ssh-rsa [EXISTING_KEY_VALUE_1] [USERNAME_1]`
* You can then add the file to the instance metadata:
    * `gcloud compute instance add-metadata <instance_name> --metadata-from-file ssh-keys=ssh_users.txt`
    * Now you can just ssh in without even using the gcloud sdk
    * This is only project level metadata however so only works for the specified instance
    
# Images and Snapshots

* A snapshot is a point in time copy of a persistent disk
`gcloud compute snapshots list` list all the current snapshots created from a disk
## Snapshots

### Creating a snapshot
`gcloud compute disks snapshot <name_of_disk>` snapshot is the verb here  
* Persistent disks like instances are zone based - the sdk needs to know which zone you want to use 
* Snapshots (like docker images) only write the differences from the last snapshot - they are incremental
    * This way snapshots take less time to create and they take less space

![snapshot](../../images/snapshots.PNG =700x)

* Once the snapshot has been created, we can restore it by creating a new disk based on that snapshot, before attaching that to an instance 
    * Use the snapshots tab when creating an instance to do so 

### Deleting a snapshot
`gcloud compute snapshots delete <name_of_snapshot>` to delete a specific snapshot 

## Images

`gcloud compute images list` show all our images

### Creating Images

* You can create an image from the current server 
    * Set the family name - allows us to group instances - can get latest image in that family, can also deprecate images 
    * Can create an image from multiple sources:
        * Disk
        * Snapshot
        * Image
        * Cloud Storage File
    * Choose whether to keep source disk running whilst creating (might not get all the changes if yes)
* You can then build a new instance based on that image (custom images tab next to snapshot in create instance console)

### Deleting Images
`gcloud compute images delete <image_name>`

# Summary

* We can use snapshots to create an incremental disk backup
    * Can use the snapshot to restore a disk or create a new image
* We can create images in a number of ways:
    * existing images
    * snapshots
    * cloud storage files
    * disks
* Can use images to create new instances
    * Images have families which allows us to group images
    * can use families to get the latest version of an image 

## From StackOverflow

* A snapshot reflects the contents of a persistent disk in a concrete instant in time. 
* An image is the same thing, but includes an operating system and boot loader and can be used to boot an instance.

* Images and snapshots can be public or private. 
    * In the case of images, public can mean official public images provided by Google or not.

* Snapshots are stored as diffs (a snapshot is stored relative to the previous one) while images are not.       
    * Snapshots are also cheaper ($0.03 per GB/month vs $0.085 for images).

These days the two concepts are quite similar. It's now possible to start an instance using a snapshot instead of an image, which is an easy way of resizing your boot partition. Using snapshots may be simpler for most cases.