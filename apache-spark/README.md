# Apache Spark Cheatsheet

Quick reference for RDDs, DataFrames, SQL, Streaming & Performance Tuning — Spark 3.x.

## 🚀 Getting Started: SparkContext & SparkSession

Initialize Spark (Python)

```python
from pyspark import SparkContext
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MyApp") \
    .master("local[4]") \
    .getOrCreate()

sc = spark.sparkContext
```

Initialize Spark (Scala)

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("MyApp")
  .master("local[4]")
  .getOrCreate()

val sc = spark.sparkContext
```

> **Tip:** Use `SparkSession` (Spark 2.0+) instead of `SparkContext`. It's unified and handles both RDDs and DataFrames.

Master URL Reference

| Master URL | Description |
|---|---|
| `local` | Single thread |
| `local[4]` | 4 local threads (good for testing) |
| `spark://host:7077` | Spark standalone cluster |
| `yarn` | YARN cluster (Hadoop) |
| `mesos://host:5050` | Mesos cluster |
| `k8s://https://host:6443` | Kubernetes cluster |

## 🧱 RDD Operations: The Foundation

Create RDDs

| Method | Description | Example |
|---|---|---|
| `sc.parallelize()` | Parallelize collection | `rdd = sc.parallelize([1,2,3,4,5])` |
| `sc.textFile()` | Read text file | `rdd = sc.textFile("/path/file.txt")` |
| `sc.range()` | Range of numbers | `rdd = sc.range(1, 100)` |
| `sc.hadoopFile()` | Read Hadoop file | `rdd = sc.hadoopFile("/hdfs/path")` |
| `rdd.map(f)` | Transform each element | `rdd2 = rdd.map(lambda x: x*2)` |

RDD Transformations

| Transformation | Output | Description |
|---|---|---|
| `map(f)` | RDD | Apply function to each element |
| `flatMap(f)` | RDD | Map then flatten results |
| `filter(f)` | RDD | Keep elements where f(x) is true |
| `distinct()` | RDD | Remove duplicates (expensive) |
| `union(other)` | RDD | Combine two RDDs |
| `join(other)` | RDD | Inner join two key-value RDDs |
| `groupByKey()` | RDD | Group values by key (slow!) |
| `reduceByKey(f)` | RDD | Reduce values per key (fast!) |
| `sortByKey()` | RDD | Sort by key |

RDD Actions

| Action | Returns | Description |
|---|---|---|
| `collect()` | List | Return all elements to driver (be careful!) |
| `count()` | Int | Number of elements |
| `first()` | Element | First element |
| `take(n)` | List | First n elements |
| `top(n)` | List | Top n elements by order |
| `foreach(f)` | None | Apply f to each element (side effects) |
| `reduce(f)` | Element | Aggregate elements |
| `saveAsTextFile()` | None | Write to file |

## 📊 DataFrames & Datasets (Recommended)

Create DataFrames

```python
df = spark.createDataFrame([(1, "Alice"), (2, "Bob")], ["id", "name"])

import pandas as pd
pdf = pd.read_csv("data.csv")
df = spark.createDataFrame(pdf)

df = spark.read.csv("/path/data.csv", header=True, inferSchema=True)
df = spark.read.parquet("/path/data.parquet")
df = spark.read.json("/path/data.json")

df = rdd.toDF(["col1", "col2"])
```

DataFrame Operations

```python
df.select("name", "age")
df.filter(df["age"] > 25)
df.where("age > 25")
df.groupBy("department").agg({"salary": "mean", "count": "count"})
df.orderBy("age", ascending=False)
df.distinct()
df1.join(df2, "id", how="inner")
df1.union(df2)
df.withColumn("new_col", df["age"] + 1)
```

DataFrame Aggregations

| Function | Description |
|---|---|
| `count()` | Number of rows |
| `sum(col)` | Sum of column |
| `mean(col)` | Average |
| `min(col)` | Minimum |
| `max(col)` | Maximum |
| `stddev(col)` | Standard deviation |
| `approx_percentile()` | Percentile (approximate) |

## 🗃️ Spark SQL: SQL on DataFrames

Register & Query

```python
df.createOrReplaceTempView("users")
df.createOrReplaceGlobalTempView("global_users")

result = spark.sql("""
    SELECT name, age, COUNT(*) as count
    FROM users
    WHERE age > 25
    GROUP BY name, age
    ORDER BY count DESC
""")
```

Common SQL Patterns

| Pattern | Example |
|---|---|
| Window Functions | `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)` |
| CTEs (Common Table Expr) | `WITH temp AS (SELECT ...) SELECT * FROM temp` |
| Subqueries | `SELECT * FROM (SELECT * FROM ...) AS t` |
| Aggregates | `GROUP BY, HAVING, aggregate functions` |
| Joins | `INNER, LEFT, RIGHT, FULL OUTER JOIN` |
| Case When | `CASE WHEN condition THEN value ELSE default END` |

## 🌊 Spark Streaming: Real-Time Processing

Streaming DataFrame (Recommended)

```python
df = spark.readStream \
    .format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic") \
    .load()

result = df.groupBy(window(df.timestamp, "10 minutes")).count()

query = result.writeStream \
    .outputMode("update") \
    .format("console") \
    .start()

query.awaitTermination()
```

Streaming Output Modes

| Mode | Description |
|---|---|
| `append` | Only new rows added since last trigger |
| `update` | All rows that changed since last trigger |
| `complete` | Entire output (for aggregations) |

## ⚙️ Configuration & Performance Tuning

Spark Configuration

| Config | Description | Default |
|---|---|---|
| `spark.driver.memory` | Driver memory (increase if OOM) | `1g` |
| `spark.executor.memory` | Memory per executor | `1g` |
| `spark.executor.cores` | Cores per executor | `1` |
| `spark.sql.shuffle.partitions` | Partitions for shuffle (increase if data large) | `200` |
| `spark.default.parallelism` | Default partitions for RDD ops | `cores x 2` |
| `spark.broadcast.blockSize` | Broadcast variable size | `4m` |
| `spark.sql.adaptive.enabled` | Adaptive query execution | `true` (3.2+) |

Set Configuration

```python
spark.conf.set("spark.sql.shuffle.partitions", "1000")
```

```bash
spark-submit --conf spark.executor.memory=4g --conf spark.executor.cores=4
# spark.executor.memory 4g
# spark.executor.cores 4
```

Performance Tuning Checklist

- **Partition count** — `spark.sql.shuffle.partitions = executors x cores x 2`
- **Memory allocation** — Driver + executor memory adequate?
- **Caching** — `df.cache()` for repeated use
- **Broadcast joins** — For small DataFrame joins
- **Predicate pushdown** — Filter early (before join)
- **Columnar storage** — Parquet, ORC more efficient than CSV

## 🚢 Spark Job Submission

spark-submit Command

```bash
spark-submit --class com.example.App app.jar

spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 2g \
  --executor-memory 4g \
  --executor-cores 4 \
  --num-executors 10 \
  --conf spark.sql.shuffle.partitions=1000 \
  app.jar arg1 arg2

spark-submit --master local[4] script.py

spark-submit \
  --master k8s://https://kubernetes-host:6443 \
  --deploy-mode cluster \
  --executor-memory 4g \
  app.jar
```

Submit Modes

| Flag | Description |
|---|---|
| `--master local[n]` | Local mode with n threads (testing) |
| `--master spark://host:7077` | Spark standalone cluster |
| `--master yarn` | YARN cluster (Hadoop) |
| `--deploy-mode client` | Driver runs on client machine |
| `--deploy-mode cluster` | Driver runs on cluster |

## 🧩 Common Patterns & Best Practices

Pattern 1: Map-Reduce with Spark

```python
result = df.rdd \
    .map(lambda row: (row.key, row.value)) \
    .reduceByKey(lambda a, b: a + b) \
    .toDF(["key", "sum"])

# Equivalent using the DataFrame API (preferred)
result = df.groupBy("key").agg({"value": "sum"})
```

Pattern 2: Broadcast Join (Small + Large)

```python
from pyspark.sql.functions import broadcast

result = large_df.join(broadcast(small_df), "id")
```

Pattern 3: Cache for Reuse

```python
df = spark.read.csv("/data/large.csv")
df.cache()

df.count()

result1 = df.filter(df.age > 25).count()
result2 = df.groupBy("dept").count().collect()
```

Pattern 4: User-Defined Functions (UDF)

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import IntegerType

def add_one(x):
    return x + 1 if x is not None else None

udf_add_one = udf(add_one, IntegerType())
df_new = df.withColumn("new_col", udf_add_one(df.age))

# Better: Pandas UDF (vectorized, faster)
from pyspark.sql.functions import pandas_udf
import pandas as pd

def vectorized_add(s: pd.Series) -> pd.Series:
    return s + 1

vector_udf = pandas_udf(vectorized_add, IntegerType())
```

Pattern 5: Window Functions

```python
from pyspark.sql.functions import row_number, rank, dense_rank, col
from pyspark.sql.window import Window

window_spec = Window.partitionBy("dept").orderBy(df.salary.desc())

result = df.withColumn("rank", rank().over(window_spec)) \
    .filter(col("rank") <= 3)
```

## ⚠️ Common Mistakes & Solutions

| Mistake | Problem | Solution |
|---|---|---|
| Using RDDs for everything | Slow, no optimization | Use DataFrames/SQL (optimized) |
| `collect()` on large RDD | Driver OOM, slow | Use `take(n)`, write to file |
| Not caching repeated use | Recomputes every action | `df.cache()` before multiple actions |
| Too few partitions | Slow, node bottleneck | Increase partitions for shuffle |
| Too many partitions | Too many small tasks, overhead | Decrease partitions, `coalesce()` |
| Not using broadcast variables | Large lookup table joins inefficient | Use `broadcast(small_df)` for joins |
| Mutable operations | Unpredictable results | Use functional transforms |
| No monitoring | Silent failures, slow jobs | Watch Spark UI, add logging |

## 🆚 RDD vs DataFrame vs SQL: Choose the Right Tool

| Aspect | RDD | DataFrame | SQL |
|---|---|---|---|
| Speed | Slow (no optimization) | Fast (Catalyst optimizer) | Fast (optimized queries) |
| Schema | None (untyped) | Strong (typed) | Strong (typed) |
| Syntax | Functional (map/filter) | DSL (`df.select()`) | SQL |
| Learning Curve | Moderate | Easy | Easy (if you know SQL) |
| Use Case | Unstructured data, custom logic | Structured data, transformations | Analytics, aggregations |
| Recommendation | Avoid unless necessary | Preferred for most cases | Best for analytics |

> **Recommendation:** Start with DataFrames (faster, easier). Only use RDDs for unstructured data or custom partitioning. Use SQL for analytics.

## 📋 Quick Reference: Common Operations

Selection & Filtering

```python
df.select("col1", "col2")
df.filter(df.age > 25)
df.drop("col")
df.distinct()
```

Data Inspection

```python
df.show()
df.printSchema()
df.describe()
df.columns
df.dtypes
```

I/O Operations

```python
df.write.parquet("/path")
df.write.csv("/path")
df.write.mode("overwrite")
spark.read.csv("/path")
```

Aggregation

```python
df.groupBy("col").count()

df.agg(F.sum("col"))

df.groupBy("col").agg({
    "salary": "mean",
    "id": "count"
})
```

> **Memory Pro Tip:** For large DataFrames, use `df.repartition(n)` to increase partitions before expensive operations like `groupBy`.

> **Watch Out:** `collect()` brings all data to the driver. On 10GB of data, this causes OOM. Use `take(n)` or write to file instead.

---
*Source: adapted from the Apache Spark cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
