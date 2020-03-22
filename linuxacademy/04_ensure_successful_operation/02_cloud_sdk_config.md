# Cloud SDK Configurations

You can have multiple Cloud SDK configs for multiple different projects   
`gcloud config configurations create <config_name>`  - to create a new SDK configuration  
`gcloud config configurations list` - to list all the existing configs  
`gcloud config configurations activate <config_name>` - to activate a certain config   
`gcloud config list` - to see all the settings for the activated config   

You can change the settings for a config by adjusting the values whilst the config is activated. 

`gcloud config set account <email@account>` - to set the service account creds used for a config   

You can quickly switch between by using the above commands for different projects that might have different service accounts, compute instances, K8s instances etc
