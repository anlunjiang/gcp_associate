# Streaming Data Processing: Streaming Data Pipelines into Bigtable

create a bt instance - 
`gcloud beta bigtable instances create sandiego --cluster=cpb210 --cluster-zone=us-central1-b --display-name=="San Diego Freeway data" --instance-type=DEVELOPMENT`

hbase - `scan 'current_conditions', {'LIMIT' => 2}` select * from current_conditions limit 2


`scan 'current_conditions', {'LIMIT' => 10, STARTROW => '15#S#1', ENDROW => '15#S#999', COLUMN => 'lane:speed'}`