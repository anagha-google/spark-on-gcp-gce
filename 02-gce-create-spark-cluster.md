# About

This module covers creation of a Cloud Dataproc cluster, with a pre-created Dataproc Metastore Service, BigQuery Apache Spark connector, and with the optional component of Jupyter. 


## Lab Modules

| Module | Resource | 
| -- | :--- |
| 1 | [Foundational Setup](01-foundational-setup.md) |
| 2 | [Create a Spark Cluster](02-gce-create-spark-cluster.md) |
| 3 | [Submit Spark batch jobs](03-run-spark-batch-jobs.md) |
| 4 | [Spark notebooks](04-run-spark-notebooks.md) |
| 10 | [Clean up](10-clean-up.md) |

## Documentation resources

| Topic | Resource | 
| -- | :--- |
| 1 | [Cloud Dataproc landing page](https://cloud.google.com/dataproc/docs) |
| 2 | [Dataproc Metastore Service](https://cloud.google.com/dataproc-metastore/docs) |
| 3 | [Dataproc Persistent Spark History Server](https://cloud.google.com/dataproc/docs/concepts/jobs/history-server) |
| 4 | [Apache Spark BigQuery connector repository](https://github.com/GoogleCloudDataproc/spark-bigquery-connector) |
| 5 | [Apache Spark](https://spark.apache.org/docs/latest/) |


## 1. Pre-requisites

Completion of the prior module
<br>
  
<hr>

## 2. Variables

```
#Replace with base_prefix of your choice, from module 1
BASE_PREFIX="vajra"  

#Replace with your details
ORG_ID=<YOUR_LDAP>.altostrat.com                              
ORG_ID_NBR=<YOUR_ORG_ID_NBR>
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 
PROJECT_ID=<YOUR_PROJECT_ID>
PROJECT_NBR=<YOUR_PROJECT_ID_NBR>

#Your public IP address, to add to the firewall
YOUR_CIDR=<YOUR_IP_ADDRESS>/32

#General variables
LOCATION=us-central1
ZONE=us-central1-a

UMSA="$BASE_PREFIX-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com

SPARK_GCE_NM=$BASE_PREFIX-gce
PERSISTENT_HISTORY_SERVER_NM=$BASE_PREFIX-sphs

SPARK_GCE_BUCKET=$SPARK_GCE_NM-$PROJECT_ID-s
SPARK_GCE_BUCKET_FQN=gs://$SPARK_GCE_BUCKET
SPARK_GCE_TEMP_BUCKET=$SPARK_GCE_NM-$PROJECT_ID-t
SPARK_GCE_TEMP_BUCKET_FQN=gs://$SPARK_GCE_TEMP_BUCKET
PERSISTENT_HISTORY_SERVER_BUCKET_FQN=gs://$PERSISTENT_HISTORY_SERVER_NM-$PROJECT_NBR


DATAPROC_METASTORE_SERVICE_NM=$BASE_PREFIX-dpms

VPC_PROJ_ID=$PROJECT_ID        
VPC_PROJ_ID=$PROJECT_NBR  

VPC_NM=$BASE_PREFIX-vpc
SPARK_GCE_SUBNET_NM=$SPARK_GCE_NM-snet
SPARK_CATCH_ALL_SUBNET_NM=$BASE_PREFIX-misc-snet


```
  
## 3. Create a Dataproc GCE cluster

Edit the version of the BigQuery connector to reflect the latest version in this Gig repos "latest" version-
https://github.com/GoogleCloudDataproc/spark-bigquery-connector


```
gcloud dataproc clusters create $SPARK_GCE_NM \
   --service-account=$UMSA_FQN \
   --project $PROJECT_ID \
   --subnet $SPARK_GCE_SUBNET_NM \
   --region $LOCATION \
   --zone $ZONE \
   --enable-component-gateway \
   --bucket $SPARK_GCE_BUCKET \
   --temp-bucket $SPARK_GCE_TEMP_BUCKET \
   --dataproc-metastore projects/$PROJECT_ID/locations/$LOCATION/services/$DATAPROC_METASTORE_SERVICE_NM \
   --master-machine-type n1-standard-4 \
   --master-boot-disk-size 500 \
   --num-workers 3 \
   --worker-machine-type n1-standard-4 \
   --worker-boot-disk-size 500 \
   --image-version 2.0-debian10 \
   --tags $SPARK_GCE_NM \
   --optional-components JUPYTER \
   --initialization-actions gs://goog-dataproc-initialization-actions-${LOCATION}/connectors/connectors.sh \
   --metadata spark-bigquery-connector-version=0.23.2
```

You should output as follows-
```
Waiting on operation [projects/dataproc-playground-335723/regions/us-central1/operations/71bd4732-4f9e-3d3e-8f10-bc4adfbde6ba].
Waiting for cluster creation operation...
WARNING: For PD-Standard without local SSDs, we strongly recommend provisioning 1TB or larger to ensure consistently high I/O performance. See https://cloud.google.com/compute/docs/disks/performance for information on disk I/O performance.
Waiting for cluster creation operation...done.     
Created [https://dataproc.googleapis.com/v1/projects/dataproc-playground-335723/regions/us-central1/clusters/vajra-gce] Cluster placed in zone [us-central1-a].
```

Here is a pictorial overview of the cluster-

![1](images/02-01.png)   
  
<br><br>

![2](images/02-02.png)   
  
<br><br>

![3](images/02-03.png)   
  
<br><br>

![4](images/02-04.png)   
  
<br><br>

![5-1](images/02-05.png)   
  
<br><br>

![5-2](images/02-05-2.png)   
  
<br><br>

![5-3](images/02-05-3.png)   
  
<br><br>

![6](images/02-06.png)   
  
<br><br>

![7](images/02-07.png)   
  
<br><br>

![8](images/02-08.png)   
  
<br><br>

![9](images/02-09.png)   
  
<br><br>

![10](images/02-10.png)   
  
<br><br>

<hr>


## 4. SSH to cluster

```
gcloud compute ssh --zone "$ZONE" "$SPARK_GCE_NM-m"  --project $PROJECT_ID
```

The above command allows you to SSH to the master node. To SSH to the other nodes, go via the Google Compute Engine UI route to get the gcloud commands.

<br><br>

<hr>
This concludes the module. <br>

[Next Module](03-run-spark-batch-jobs.md) 
<br>
[Repo Landing Page](README.md)

<hr>
