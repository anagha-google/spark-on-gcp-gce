# About

This module covers fundamentals of running Spark on Dataproc-GCE through a practical and basic set of examples to get you quick-started. 

## Documentation resources

| Topic | Resource | 
| -- | :--- |
| 1 | [Cloud Dataproc landing page](https://cloud.google.com/dataproc/docs) |
| 2 | [Dataproc Metastore Service](https://cloud.google.com/dataproc-metastore/docs) |
| 3 | [Dataproc Persistent Spark History Server](https://cloud.google.com/dataproc/docs/concepts/jobs/history-server) |
| 4 | [Apache Spark](https://spark.apache.org/docs/latest/) |

## 1. Pre-requisites

Completion of the prior module
<br>
  
<hr>

## 2. Variables

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

SPARK_GCE_BUCKET_FQN=gs://$SPARK_GCE_NM-$PROJECT_NBR
SPARK_GCE_SCRATCH_BUCKET_FQN=gs://$SPARK_GCE_NM-$PROJECT_NBR-scratch
SPARK_GCE_BUCKET=$SPARK_GCE_NM-$PROJECT_NBR
SPARK_GCE_SCRATCH_BUCKET=$SPARK_GCE_NM-$PROJECT_NBR-scratch

PERSISTENT_HISTORY_SERVER_NM=$BASE_PREFIX-sphs
PERSISTENT_HISTORY_SERVER_BUCKET_FQN=gs://$PERSISTENT_HISTORY_SERVER_NM-$PROJECT_NBR
DATAPROC_METASTORE_SERVICE_NM=$BASE_PREFIX-dpms

VPC_PROJ_ID=$PROJECT_ID        
VPC_PROJ_ID=$PROJECT_NBR  

VPC_NM=$BASE_PREFIX-vpc
SPARK_GCE_SUBNET_NM=$SPARK_GCE_NM-snet
SPARK_CATCH_ALL_SUBNET_NM=$BASE_PREFIX-misc-snet
```
  
## 3. Create a Dataproc GCE cluster
```
gcloud dataproc clusters create $SPARK_GCE_NM \
   --service-account=$UMSA_FQN \
   --project $PROJECT_ID \
   --subnet $SPARK_GCE_SUBNET_NM \
   --region $LOCATION \
   --zone $ZONE \
   --enable-component-gateway \
   --bucket $SPARK_GCE_BUCKET_FQN \
   --temp-bucket $SPARK_GCE_SCRATCH_BUCKET_FQN \
   --dataproc-metastore projects/$PROJECT_ID/locations/$LOCATION/services/$DATAPROC_METASTORE_SERVICE_NM \
   --master-machine-type n1-standard-4 \
   --master-boot-disk-size 500 --num-workers 3 \
   --worker-machine-type n1-standard-4 \
   --worker-boot-disk-size 500 \
   --image-version 2.0-debian10 \
   --tags $SPARK_GCE_NM 

```

<hr>
