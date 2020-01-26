

## IAM Internet Access Management 

Group accounts (gmail accounts) into groups and assign roles to that account   
Policy: associate roles to member - each association is a policy 

Attach policy to google resource or a project / all child resources and project will inherit that policy  
assign policies to folders? (nuance to check)

## 3 main types of roles
* Primitive: Historical GCP roles, Avoid as can include wide range of permissions across all GCP services (Owner, Editor, Viewer) should be predefined ones like storageViewer
* Predefined: Like AD Groups, finer access control than primitive roles 
* Custom:  


# Cloud IAM Qwiklab


Buckets are globally unique - and can be multi region  
be aware of the data you're storing with the bucket and complies with GDPR

Storage Class: Latency  - nearline, archive (cold start) depends of frequency of data reads

