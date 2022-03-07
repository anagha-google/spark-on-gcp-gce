# About

This module shows how to submit jobs to a Cloud Dataproc cluster

## 1. Variables

```
#Replace with base prefix you chose in module 1
BASE_PREFIX="zeus"  

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
DATAPROC_METASTORE_SERVICE_NM=$BASE_PREFIX-dpms
SPARK_GCE_SUBNET_NM=$SPARK_GCE_NM-snet


INPUT_BUCKET_FQN=gs://vajra-gce-data/wordcount/input/crimes/Wards.csv
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

## 2. SparkPi job
```
gcloud dataproc jobs submit spark \
--cluster=${SPARK_GCE_NM} \
--region=$LOCATION \
--jars=file:///usr/lib/spark/examples/jars/spark-examples.jar \
--class org.apache.spark.examples.SparkPi -- 10000
```

## 3. Wordcount job

```
gsutil rm -R $OUTPUT_BUCKET_FQN

gcloud dataproc jobs submit spark \
    --cluster=${SPARK_GCE_NM} \
    --class=${CLASS_NAME} \
    --jars=${JAR_BUCKET_FQN}/${JAR_NAME} \
    --region=${LOCATION} \
    --impersonate-service-account $UMSA_FQN \
    -- ${INPUT_BUCKET_FQN} ${OUTPUT_BUCKET_FQN} 
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
