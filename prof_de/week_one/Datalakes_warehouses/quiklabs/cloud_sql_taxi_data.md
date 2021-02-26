# Loading Taxi Data into Google Cloud SQL

export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml

gcloud sql instances create taxi \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS


gcloud sql users set-password root --host % --instance taxi \
 --password Passw0rd


Now create an environment variable with the IP address of the Cloud Shell:
`export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32`

Whitelist the Cloud Shell instance for management access to your SQL instance.
`gcloud sql instances patch taxi --authorized-networks $ADDRESS`

get info about a sql instance:
`gcloud sql instances describe taxi (--format="value(ipAddresses.ipAddress)")`

login to mysql `mysql --host=$MYSQLIP --user=root --password --verbose`

copy from bucket to the machine hosting the cloud shell: `gsutil cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv trips.csv-1`

Import csv into mysql: `mysqlimport --local --host=$MYSQLIP --user=root --password --ignore-lines=1 --fields-terminated-by=',' bts trips.csv-*`