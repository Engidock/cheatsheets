# Big Data Cheatsheet

Complete quick reference guide covering Hadoop, Spark, Kafka, Hive, SQL analytics, cloud big data services, file formats, NoSQL stores, orchestration, security, and performance tuning.

## 🐘 Hadoop Ecosystem

### HDFS Commands

List files in a directory:
```bash
hdfs dfs -ls /path
```

Create a directory (with parents):
```bash
hdfs dfs -mkdir -p /path/to/dir
```

Upload a local file to HDFS:
```bash
hdfs dfs -put local.txt /hdfs/path
```

Download a file from HDFS:
```bash
hdfs dfs -get /hdfs/path local.txt
```

View file contents:
```bash
hdfs dfs -cat /path/file.txt
```

Remove a file or directory recursively:
```bash
hdfs dfs -rm -r /path
```

Show disk usage in human-readable form:
```bash
hdfs dfs -du -h /path
```

Copy files within HDFS:
```bash
hdfs dfs -cp /src /dest
```

Move/rename files within HDFS:
```bash
hdfs dfs -mv /src /dest
```

Change file permissions:
```bash
hdfs dfs -chmod 755 /path
```

Cluster health report:
```bash
hdfs dfsadmin -report
```

Check filesystem consistency and block info:
```bash
hdfs fsck /path -files -blocks
```

Rebalance data across DataNodes:
```bash
hdfs balancer
```

### YARN Commands

List running applications:
```bash
yarn application -list
```

Kill an application:
```bash
yarn application -kill app_id
```

Check application status:
```bash
yarn application -status app_id
```

Fetch application logs:
```bash
yarn logs -applicationId app_id
```

List nodes in the cluster:
```bash
yarn node -list
```

## ⚡ Apache Spark

### Spark Submit

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 4g \
  --executor-memory 8g \
  --executor-cores 4 \
  --num-executors 10 \
  app.py
```

### PySpark DataFrame API

Reading data:
```python
df = spark.read.parquet("path")

df = spark.read.csv("path", header=True, inferSchema=True)

df = spark.read.json("path")

df = spark.table("database.table")
```

Inspecting data:
```python
df.show(20, truncate=False)

df.printSchema()

df.count()

df.columns

df.dtypes
```

Transformations:
```python
df.select("col1", "col2")

df.filter(col("age") > 25)

df.where(col("country") == "US")

df.groupBy("category").agg(count("*"), avg("price"))

df.orderBy(col("date").desc())

df.limit(100)

df.distinct()

df.dropDuplicates(["id"])

df.drop("col1", "col2")

df.withColumn("new_col", col("old_col") * 2)

df.withColumnRenamed("old", "new")

df1.join(df2, "key", "inner")

df1.join(df2, df1.id == df2.id, "left")

df1.join(broadcast(df2), "key")
```

Aggregations:
```python
from pyspark.sql.functions import *

df.groupBy("category").agg(
    count("*").alias("total"),
    sum("amount").alias("sum_amount"),
    avg("price").alias("avg_price"),
    max("date").alias("latest_date")
)
```

Window functions:
```python
from pyspark.sql.window import Window

window = Window.partitionBy("category").orderBy("date")

df.withColumn("rank", row_number().over(window))

df.withColumn("running_total", sum("amount").over(window))
```

Writing data:
```python
df.write.parquet("path", mode="overwrite")

df.write.partitionBy("date").parquet("path")

df.write.bucketBy(100, "id").saveAsTable("table")

df.write.format("delta").save("path")
```

### Spark SQL

```python
spark.sql("""
    SELECT
        category,
        COUNT(*) as total,
        SUM(amount) as revenue
    FROM sales
    WHERE date >= '2024-01-01'
    GROUP BY category
    HAVING revenue > 10000
    ORDER BY revenue DESC
""").show()
```

Register a temp view:
```python
df.createOrReplaceTempView("temp_table")
```

User-defined functions (UDFs):
```python
from pyspark.sql.types import StringType

def upper_case(s):
    return s.upper()

spark.udf.register("upper_udf", upper_case, StringType())

spark.sql("SELECT upper_udf(name) FROM table")
```

### Spark Performance Tuning

**Best practices — avoid:**
- Too many partitions (<10MB each)
- Too few partitions (underutilized cluster)
- Using `collect()` on large datasets
- Shuffling before filtering
- Not caching iterative computations
- `SELECT *` on wide tables
- Avoid UDFs (use built-in functions)

**Best practices — do:**
- Use `coalesce()` to reduce partitions (no shuffle)
- Cache DataFrames accessed multiple times
- Broadcast small tables (<10MB)
- Partition data by frequently filtered columns
- Use Parquet with Snappy compression
- Enable Adaptive Query Execution (AQE)

**Key configuration parameters:**

| Parameter | Purpose | Recommended Value |
|---|---|---|
| `spark.sql.shuffle.partitions` | Partitions after shuffle | 200 (2-3x cores) |
| `spark.default.parallelism` | Default RDD partitions | 2-3x total cores |
| `spark.sql.adaptive.enabled` | Enable runtime optimization (AQE) | `true` |
| `spark.sql.autoBroadcastJoinThreshold` | Auto broadcast threshold | 10MB |
| `spark.executor.memory` | Executor heap memory | 4-32GB |
| `spark.executor.cores` | Cores per executor | 4-6 |

## 📨 Apache Kafka

### Kafka CLI Commands

**Topics:**
```bash
# List topics
kafka-topics --bootstrap-server localhost:9092 --list

# Create a topic
kafka-topics --create --topic my-topic --partitions 10 --replication-factor 3

# Describe a topic
kafka-topics --describe --topic my-topic

# Delete a topic
kafka-topics --delete --topic my-topic
```

**Producer:**
```bash
kafka-console-producer --broker-list localhost:9092 --topic my-topic

kafka-console-producer --broker-list localhost:9092 --topic my-topic \
  --property "key.separator=:" --property "parse.key=true"
```

**Consumer:**
```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic my-topic --from-beginning

kafka-console-consumer --bootstrap-server localhost:9092 --topic my-topic --group my-group

kafka-console-consumer --bootstrap-server localhost:9092 --topic my-topic --property print.key=true
```

**Consumer groups:**
```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --list

kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group

kafka-consumer-groups --bootstrap-server localhost:9092 --reset-offsets --group my-group \
  --topic my-topic --to-earliest --execute
```

### Kafka Python Producer/Consumer

```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)
producer.send('my-topic', {'key': 'value'})
producer.flush()
```

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest',
    group_id='my-group',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    print(message.value)
```

## 🐝 Hive

### Hive DDL

```sql
CREATE DATABASE IF NOT EXISTS mydb;
USE mydb;

CREATE TABLE users (
    user_id INT,
    name STRING,
    email STRING,
    created_date DATE
)
PARTITIONED BY (country STRING, year INT)
STORED AS PARQUET;

CREATE EXTERNAL TABLE logs (
    log_id STRING,
    message STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/data/logs';

ALTER TABLE users ADD COLUMNS (phone STRING);
ALTER TABLE users ADD PARTITION (country='US', year=2024);
ALTER TABLE users DROP PARTITION (country='US', year=2023);

SHOW DATABASES;
SHOW TABLES;

DESCRIBE users;
DESCRIBE FORMATTED users;
SHOW PARTITIONS users;
```

### Hive DML

```sql
INSERT INTO TABLE users PARTITION (country='US', year=2024)
SELECT user_id, name, email, created_date FROM staging;

INSERT OVERWRITE TABLE users PARTITION (country='US', year=2024)
SELECT * FROM staging WHERE country='US' AND YEAR(date)=2024;

SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;

INSERT INTO TABLE users PARTITION (country, year)
SELECT *, country, year FROM staging;
```

## 📊 SQL Essentials

### Window Functions

```sql
-- Ranking
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num,
    RANK() OVER (ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank,
    NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;
```

```sql
-- Running totals / moving averages
SELECT
    employee_id,
    date,
    amount,
    SUM(amount) OVER (PARTITION BY employee_id ORDER BY date) as running_total,
    AVG(amount) OVER (PARTITION BY employee_id ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg,
    LAG(amount, 1) OVER (PARTITION BY employee_id ORDER BY date) as prev_amount,
    LEAD(amount, 1) OVER (PARTITION BY employee_id ORDER BY date) as next_amount
FROM sales;
```

### Common SQL Patterns

Pivot with conditional aggregation:
```sql
SELECT
    product,
    SUM(CASE WHEN year=2023 THEN sales ELSE 0 END) as sales_2023,
    SUM(CASE WHEN year=2024 THEN sales ELSE 0 END) as sales_2024
FROM sales_data
GROUP BY product;
```

Cumulative sum:
```sql
SELECT
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as cumulative
FROM transactions;
```

Find gaps in a sequence:
```sql
WITH gaps AS (
    SELECT
        id,
        id - LAG(id) OVER (ORDER BY id) as gap
    FROM sequence_table
)
SELECT * FROM gaps WHERE gap > 1;
```

## ☁️ Cloud Big Data Services

| Category | AWS | Azure | GCP |
|---|---|---|---|
| Managed Hadoop/Spark | EMR: Managed Spark | Databricks: Unified analytics | Dataproc: Managed Hadoop/Spark |
| Serverless SQL on object storage | Athena: Serverless SQL on S3 | Synapse: Analytics service | BigQuery: Serverless DW |
| ETL service + catalog | Glue: ETL service + catalog | Data Factory: ETL/ELT | Dataflow: Apache Beam |
| Real-time streaming / messaging | Kinesis: Streaming | Event Hub: Messaging | Pub/Sub: Real-time streaming |
| Data warehouse | Redshift: Data warehouse | HDInsight: Hadoop/Spark | BigQuery: Data warehouse |
| Object / data lake storage | S3: Object storage | ADLS: Data lake storage | GCS: Object storage |
| Serverless compute | Lambda: Serverless compute | Functions: Serverless | Cloud Run / Cloud Functions: Serverless |
| Orchestration | MWAA: Managed Airflow | Data Factory: Pipelines | Cloud Composer: Airflow |

### BigQuery

```sql
CREATE TABLE dataset.sales
PARTITION BY DATE(order_date)
CLUSTER BY customer_id, region AS
SELECT * FROM source_table;

-- Time travel query
SELECT * FROM dataset.sales FOR SYSTEM_TIME AS OF '2024-01-01';

-- Create and train an ML model
CREATE OR REPLACE MODEL dataset.churn_model
OPTIONS (model_type='logistic_reg', input_label_cols=['churned']) AS
SELECT * FROM dataset.training_data;

-- Predict with the model
SELECT * FROM ML.PREDICT(MODEL dataset.churn_model, TABLE dataset.new_customers);
```

## 📁 File Formats & Compression

| Format | Layout | Compression | Best For |
|---|---|---|---|
| Parquet | Columnar | Snappy, Gzip | Analytics, SQL queries |
| ORC | Columnar | Zlib | Hive, optimized reads |
| Avro | Row-based | Snappy | Schema evolution, streaming |
| JSON | Row-based | Gzip | APIs, semi-structured data |
| CSV | Row-based | Gzip | Data exchange |

## 🗄️ NoSQL Databases

### HBase Shell

```bash
create 'users', 'info', 'activity'

list

describe 'users'

disable 'users'

drop 'users'

put 'users', 'row1', 'info:name', 'John'

get 'users', 'row1'

scan 'users'

scan 'users', {STARTROW => 'row1', ENDROW => 'row5'}

delete 'users', 'row1', 'info:name'
```

### MongoDB

```javascript
db.users.insertOne({name: "John", age: 30})
db.users.insertMany([{name: "Jane"}, {name: "Bob"}])

db.users.find({age: {$gt: 25}})
db.users.find({}, {name: 1, _id: 0})
db.users.findOne({name: "John"})

db.users.updateOne({name: "John"}, {$set: {age: 31}})
db.users.updateMany({age: {$lt: 18}}, {$set: {minor: true}})

db.orders.aggregate([
    {$match: {status: "completed"}},
    {$group: {_id: "$customer", total: {$sum: "$amount"}}},
    {$sort: {total: -1}},
    {$limit: 10}
])
```

### Cassandra CQL

```sql
CREATE KEYSPACE myks WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
USE myks;

CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP
);

INSERT INTO users (user_id, name, email) VALUES (uuid(), 'John', 'john@email.com');
SELECT * FROM users WHERE user_id = 123e4567-e89b-12d3-a456-426614174000;
```

## 🔄 Workflow Orchestration (Airflow)

### DAG Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'etl_pipeline',
    default_args=default_args,
    schedule_interval='0 2 * * *',
    catchup=False
)

def extract_data():
    print("Extracting data...")

def transform_data():
    print("Transforming data...")

extract = PythonOperator(task_id='extract', python_callable=extract_data, dag=dag)
transform = PythonOperator(task_id='transform', python_callable=transform_data, dag=dag)
load = BashOperator(task_id='load', bash_command='spark-submit load.py', dag=dag)

extract >> transform >> load
```

### Airflow CLI

```bash
airflow dags list

airflow dags trigger my_dag

airflow dags pause my_dag

airflow dags unpause my_dag

airflow tasks test my_dag task_id 2024-01-01

airflow dags backfill my_dag -s 2024-01-01 -e 2024-01-31
```

## 🔐 Security & Governance

### Kerberos Authentication

```bash
kinit -kt user.keytab user@REALM

klist

kdestroy

spark-submit --principal user@REALM --keytab user.keytab script.py
```

### Data Masking Patterns

| Data Type | Raw Value | Masked |
|---|---|---|
| SSN | 123-45-6789 | XXX-XX-6789 |
| Email | john@email.com | j***@email.com |
| Credit Card | 4532-1234-5678-9010 | XXXX-XXXX-XXXX-9010 |
| Phone | +1-555-123-4567 | +1-555-XXX-XXXX |

## 🚀 Performance Optimization

### Quick Optimization Checklist

- Filter early (predicate pushdown)
- Select only needed columns
- Use bucketing for joins
- Enable Adaptive Query Execution
- Monitor data skew
- Use columnar formats (Parquet/ORC)
- Partition by frequently filtered columns
- Compress with Snappy (balance)
- Cache intermediate results
- Broadcast small tables (<10MB)
- Coalesce to reduce partitions
- Avoid UDFs (use built-ins)

### Data Skew Solutions

Salting technique to distribute skewed keys across partitions:
```python
from pyspark.sql.functions import rand, concat, lit

df_salted = df.withColumn("salted_key",
    concat(col("customer_id"), lit("_"), (rand() * 10).cast("int")))

small_salted = small_df.crossJoin(
    spark.range(10).select(col("id").alias("salt"))
).withColumn("salted_key",
    concat(col("customer_id"), lit("_"), col("salt")))
```

## 🔍 Monitoring & Debugging

### Spark UI Analysis

| Symptom | Signal | Fix |
|---|---|---|
| Task Duration | Max >> Median (10x+) | Data skew — use salting |
| Shuffle Read/Write | Large shuffle size (TB+) | Broadcast small tables |
| GC Time | > 10% of task time | Increase executor memory |
| Spill (Memory) | Frequent spills to disk | Reduce cores or increase memory |
| Stage Count | Too many stages (50+) | Reduce shuffles, coalesce |

### Common Errors & Solutions

| Error | Solutions |
|---|---|
| `OutOfMemoryError` | Increase executor memory; check for data skew; increase shuffle partition count; use more partitions; optimize slow transformations; avoid `collect()` on large data |
| `TimeoutException` | Increase network timeout config; reduce executor cores; increase memory overhead; scale cluster resources |

## ✅ Best Practices Summary

- Filter early, select specific columns
- Parquet for analytics
- Partition by date/region
- Version control pipelines
- Automated testing (CI/CD)
- Cache reused DataFrames
- Monitoring and alerting
- Data quality checks
- 128MB-1GB file sizes
- Broadcast small tables
- Snappy compression
- Avoid shuffles when possible
- Monitor and optimize skew
- Documentation and lineage
- Schema evolution support

## 📋 Quick Reference Table

| Use Case | Recommended Tools | Notes |
|---|---|---|
| Batch ETL | Spark | TB-PB scale, complex transformations |
| Real-time Streaming | Kafka + Flink/Spark | Low latency, high throughput events |
| SQL Analytics | Hive, Presto, BigQuery | Ad-hoc queries, BI dashboards |
| Workflow Orchestration | Airflow | Complex DAGs, scheduling, monitoring |
| Data Warehouse | Redshift, Snowflake, BigQuery | Structured data, fast queries, BI |
| NoSQL Storage | HBase, Cassandra, MongoDB | Key-value, low-latency lookups |
| Object Storage | S3, ADLS, GCS | Data lake, cheap long-term storage |
| ML Training | Spark MLlib, TensorFlow | Distributed model training |

---
*Source: adapted from the Big Data cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
