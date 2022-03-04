# About

This module shows how to submit a batch job to a Cloud Dataproc cluster

## 1. Submit job to a cluster

```
BASE_PREFIX="vajra"  

#Replace with your details
ORG_ID=akhanolkar.altostrat.com                              
ORG_ID_NBR=236589261571
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 
PROJECT_ID=dataproc-playground-335723
PROJECT_NBR=481704770619

#Your public IP address, to add to the firewall
YOUR_CIDR=98.222.97.10/32

#General variables
LOCATION=us-central1
ZONE=us-central1-a

UMSA="$BASE_PREFIX-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com

SPARK_GCE_NM=$BASE_PREFIX-gce
DATAPROC_METASTORE_SERVICE_NM=$BASE_PREFIX-dpms


INPUT_BUCKET_FQN=gs://vajra-gce-data/input/crimes/Wards.csv
OUTPUT_BUCKET_FQN=gs://vajra-gce-data/output/scala-wordcount-output
JAR_BUCKET_FQN=gs://vajra-gce-jar/wordcount
JAR_NAME=readgcsfile_2.12-0.1.jar
CLASS_NAME=ReadGCSFileAndWordCount
```

One time activity-
```
gsutil rm -R $OUTPUT_BUCKET_FQN
gsutil rm -R $JAR_BUCKET_FQN
gsutil cp "/Users/akhanolkar/IdeaProjects/ReadGCSFile/target/scala-2.12/readgcsfile_2.12-0.1.jar" ${JAR_BUCKET_FQN}/
```

Submit job-
```
gcloud dataproc jobs submit spark \
    --cluster=${CLUSTER} \
    --class=${CLASS_NAME} \
    --jars=${JAR_BUCKET_FQN}/${JAR_NAME} \
    --region=${REGION} \
    -- ${INPUT_BUCKET_FQN} ${OUTPUT_BUCKET_FQN}
```
