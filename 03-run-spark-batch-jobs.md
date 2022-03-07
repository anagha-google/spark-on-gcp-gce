# About

This module shows how to submit jobs to a Cloud Dataproc cluster

## 1. Variables

```
#Replace with base prefix you chose in module 1
BASE_PREFIX="vajra"  

#Replace with your details
ORG_ID=<YOUR_LDAP>.altostrat.com                              
ORG_ID_NBR=<YOUR_ORG_ID_NBR>
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 
PROJECT_ID=<YOUR_PROJECT_ID>
PROJECT_NBR=<YOUR_PROJECT_ID_NBR>

#Your public IP address, to add to the firewall
YOUR_CIDR=<YOUR_IP_ADDRESS>/32


#................................

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
SPARK_GCE_SUBNET_NM=$SPARK_GCE_NM-snet

DATA_BUCKET_FQN=gs://$BASE_PREFIX-gce-data
JAR_BUCKET_FQN=gs://$BASE_PREFIX-gce-jar

JAR_NAME=readgcsfile_2.12-0.1.jar
CLASS_NAME=ReadGCSFileAndWordCount
```


## 2. Run a "SparkPi" job
This job merely calculates the value of Pi and emits the result to the screen and is great for a basic environment setup set.

```
gcloud dataproc jobs submit spark \
--cluster=${SPARK_GCE_NM} \
--region=$LOCATION \
--jars=file:///usr/lib/spark/examples/jars/spark-examples.jar \
--class org.apache.spark.examples.SparkPi -- 10000
```

In the output you should see-
```
Pi is roughly 3.141681127141681
```

Navigate to the Dataproc "batch job" UI on the Cloud Console and explore the batch job UI and logs.


## 3. Run a "Wordcount" job

### 3.1. Create storage buckets (one time activity)

#### 3.1.a. Create a bucket for the source file
```
gsutil mb -p $PROJECT_ID -c STANDARD -l $LOCATION -b on $DATA_BUCKET_FQN
```

#### 3.1.b. Create a bucket for the "Wordcount" jar (one time activity)
```
gsutil mb -p $PROJECT_ID -c STANDARD -l $LOCATION -b on $JAR_BUCKET_FQN
```

## 3.2. Copy the source data and jar to the buckets created

### 3.2.a. Clone this git repo via gcloud in Cloud Shell

```
cd ~
git clone https://github.com/anagha-google/spark-on-gcp-gce.git
```

### 3.2.b. Copy the source file to the source bucket

```
cd ~/spark-on-gcp-gce/
gsutil cp data/Wards.csv $DATA_BUCKET_FQN/input/wordcount/crimes/Wards.csv
```

### 3.2.c. Copy the jar file to the jar bucket

```
cd ~/spark-on-gcp-gce/
gsutil cp jars/readgcsfile_2.12-0.1.jar $JAR_BUCKET_FQN/wordcount/
```

## 3.3. Submit the "Wordcount" job as the service account

```
#Clean up output from potential prior runs
gsutil rm -R $DATA_BUCKET_FQN/output/wordcount 

#Submit job
gcloud dataproc jobs submit spark \
    --cluster=${SPARK_GCE_NM} \
    --class=${CLASS_NAME} \
    --jars=$JAR_BUCKET_FQN/wordcount/readgcsfile_2.12-0.1.jar \
    --region=${LOCATION} \
    --impersonate-service-account $UMSA_FQN \
    -- ${DATA_BUCKET_FQN}/input/wordcount/crimes/Wards.csv ${DATA_BUCKET_FQN}/output/wordcount 
```

1. Navigate to the Dataproc UI, to the "job" GUI and view the execution logs<br>


2. Navigate to the GCS bucket for output and view the files created there. <br>
You can also review the files via gcloud command-
```
gsutil ls ${DATA_BUCKET_FQN}/output/wordcount 
```


## 4. Notebook 

```
# 1) Create or find a GCS bucket which your cluster can access.
# 2) Prepare a CSV file of format INT,STRING
#    at gs://<bucket>/hive-warehouse/my_table/data.csv
#    You can use https://github.com/apache/spark/blob/v3.1.2/examples/src/main/resources/kv1.txt

from pyspark.sql import SparkSession
from pyspark.sql import Row

# Create a Spark session.
spark = SparkSession \
    .builder \
    .appName("SME demo") \
    .enableHiveSupport() \
    .getOrCreate()

spark
```

```
# Create a Hive external table which is backed by existing GCS files.
spark.sql("DROP TABLE IF EXISTS my_table")
spark.sql(
    "CREATE EXTERNAL TABLE IF NOT EXISTS my_table (key STRING, value INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' "
    + "LOCATION 'gs://vajra-gce-data/hive-warehouse/my_table'")
```

```
# Verify the table was created successfully.
spark.sql("SELECT * FROM my_table").show(10)
```

```
# Do some computation with the data.
results = spark.sql("SELECT key, count(value) as sum FROM my_table GROUP BY key ORDER BY key")

# Register the results as a temp view.
results.createOrReplaceTempView("results")
```

```
# Create an output table in Hive.
spark.sql("DROP TABLE IF EXISTS output_table")
spark.sql("CREATE TABLE IF NOT EXISTS output_table (key STRING, value INT) USING hive "
          + "LOCATION 'gs://vajra-gce-data/hive-warehouse/output_table'")      
```

```
# Save the results in the output table.
spark.sql("INSERT OVERWRITE output_table select * from results")
```

```
# Verify the results are saved.
spark.sql("SELECT * FROM output_table").show(10)
```

```
# Show tables in Hive metastore.
spark.sql("show tables").show()
```
