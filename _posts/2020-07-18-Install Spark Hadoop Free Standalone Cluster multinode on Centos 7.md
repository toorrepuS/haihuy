---
layout: post
title: Install Spark Hadoop Free Standalone Cluster Multinode on Centos 7
---

## Prerequisite

Below is the requirement items that needs to be installed/setup before installing Spark
- Passwordless SSH
- Java Installation and JAVA_HOME declaration in .bashrc
- Hadoop Multinode installed and start up
- Hostname/hosts config

### Download and Install Spark

On all nodes, do the followings


```python
# Create Spark directory
mkdir spark
```


```python
# On Spark Download website, select Spark with user provided Hadoop and download the binary version
wget http://mirrors.viethosting.com/apache/spark/spark-2.4.5/spark-2.4.5-bin-without-hadoop.tgz
```


```python
# Untar the file
tar -zxf spark-2.4.5-bin-without-hadoop.tgz
```

### Setup Spark Environment Variable

On all nodes, do the following


```python
# Edit the .bashrc file
nano ~/.bashrc
```


```python
# Add the following line to the file
# Set Spark Environment Variable
export SPARK_HOME=/home/odyssey/spark/spark-2.4.5-bin-without-hadoop
export PYSPARK_PYTHON=python3
export PYSPARK_DRIVER_PYTHON="jupyter" 
export PYSPARK_DRIVER_PYTHON_OPTS="notebook --no-browser --port=8889" 
export SPARK_LOCAL_IP="10.56.237.195"

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
export SPARK_DIST_CLASSPATH=$HADOOP_HOME/etc/hadoop
export SPARK_DIST_CLASSPATH=($HADOOP_HOME/bin/hadoop classpath)
```


```python
# In spark/conf directory, make a copy of spark-env.sh.template and name it as spark-env.sh
cp spark-env.sh.template spark-env.sh
```


```python
# Edit the spark-env.sh 
nano spark-env.sh
# Add below line to the file
export SPARK_DIST_CLASSPATH=$(${HADOOP_HOME}/bin/hadoop classpath)
```


```python
# In spark/conf directory, make a copy of spark-defaults.conf.template and name it as spark-defaults.conf
cp spark-defaults.conf.template spark-defaults.conf
```


```python
# Edit the spark-defaults.conf
nano spark-defaults.conf
# Add below line to the file
spark.jars.packages org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.5
spark.driver.extraJavaOptions -Dhttp.proxyHost=10.56.224.31 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=10.56.224.31 -Dhttps.proxyPort=3128
```


```python
# Copy the Hadoop config files to spark/conf directory
cp $HADOOP_HOME/etc/hadoop/core-site.xml $SPARK_HOME/conf
cp $HADOOP_HOME/etc/hadoop/hdfs-site.xml $SPARK_HOME/conf
cp $HADOOP_HOME/etc/hadoop/mapred-site.xml $SPARK_HOME/conf
cp $HADOOP_HOME/etc/hadoop/yarn-site.xml $SPARK_HOME/conf
```

# Setup Spark Cluster Manager

## 1. Spark Standalone Cluster Manager

### 1.1. Setup Spark Master Node

On Spark Master node, do

In spark/conf directory, make a copy of slaves.template and name it as slaves<br/>
`cp slaves.template slaves`

Edit the spark-defaults.conf  
`nano slaves`  
Add below line to the file  
`slaves1` # hostname of node2  
`slaves2` # hostname of node3

### 1.2. Start Spark Standalone Cluster mode


```python
# Execute the following line
$SPARK_HOME/sbin/start-all.sh
# Run jps to check if Spark is running on the Master and Slave Node 
jps
# To stop Spark Standalone Cluster mode, do
$SPARK_HOME/sbin/stop-all.sh
```


```python
# Spark Standalone Cluster mode Web UI
master:8080
# Spark Application Web UI
master:4040
# Master IP and port for spark-submit
master:7077
```

### Submit jobs to Spark Submit


```python
$SPARK_HOME/bin/spark-submit --
```


```python
# Example
# Run Mobile App Schemaless Consumer
$SPARK_HOME/bin/spark-submit --conf "spark.driver.extraJavaOptions=-Dhttp.proxyHost=10.56.224.31 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=10.56.224.31 -Dhttps.proxyPort=3128" --packages org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.4 --master spark://10.56.237.195:7077 --deploy-mode client /home/odyssey/projects/mapp/kafka_2_hadoop/Schemaless_Structure_Streaming.py

# Run HDFS dump data
$SPARK_HOME/bin/spark-submit --master spark://10.56.237.195:7077 --deploy-mode client /home/odyssey/projects/mapp/dump/hdfs_sstreaming_dump_1_1.py

# Run Kafka dump data
$SPARK_HOME/bin/spark-submit --conf "spark.driver.extraJavaOptions=-Dhttp.proxyHost=10.56.224.31 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=10.56.224.31 -Dhttps.proxyPort=3128" --packages org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.5 --master spark://10.56.237.195:7077 --deploy-mode client /home/odyssey/projects/mapp/dump/kafka_sstreaming_dump_1_1.py

# Run Schema Inferno
$SPARK_HOME/bin/spark-submit --master spark://10.56.237.195:7077 --deploy-mode client /home/odyssey/projects/mapp/schema_inferno/Schema_Saver.py
```


```python
# Note
kafka dependencies might not work for version 2.12. Downgrade to 2.11 
org.apache.spark:spark-sql-kafka-0-10_2.12:2.4.5 => org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.5
```

### Spark Application Structure


```python
# Python
import os
from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession, SQLContext
from pyspark.sql import functions as F
from pyspark.sql.types import *


def asdf(path):
    print ...

if __name__ == '__main__':
    # Setup Spark Config
    conf = SparkConf()
    conf.set("spark.executor.memory", "2g")
    conf.set("spark.executor.cores", "2")
    conf.set("spark.cores.max", "8")
    conf.set("spark.driver.extraClassPath", "/opt/oracle-client/instantclient_19_6/ojdbc8.jar")
    conf.set("spark.executor.extraClassPath", "/opt/oracle-client/instantclient_19_6/ojdbc8.jar")
    sc = SparkContext(master="spark://10.56.237.195:7077", appName="Schema Inferno", conf=conf)
    spark = SparkSession(sc)
    sqlContext = SQLContext(sc)  

    # Generate Schema
    path = 'hdfs://master:9000/home/odyssey/data/raw_hdfs'
    temp_df = spark.read.json(f'{path}/*/part-00000')
    temp_df.printSchema()
    print("Number of records: ", temp_df.count())
    print("Number of distinct records: ", temp_df.distinct().count())
    #print(temp_df.filter('payload is null').count())
    save_schema(path)
```

### Log Print Control


```python
# Create the log4j file from the template in spark/conf directory
cp conf/log4j.properties.template conf/log4j.properties
```


```python
# Replace this line 
log4j.rootCategory=INFO, console
# By this
log4j.rootCategory=WARN, console
```

### Spark Default Config


```python

```
