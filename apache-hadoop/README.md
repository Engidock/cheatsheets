# Apache Hadoop Cheatsheet

Quick reference guide for HDFS, MapReduce, YARN, and best practices.

## 🗂️ HDFS Architecture & Quick Facts

| Component | Role | Key Details |
|---|---|---|
| NameNode | Master (metadata) | File system namespace, file-to-block mapping. SPOF (use HA) |
| DataNode | Slave (storage) | Block storage, heartbeat to NN every 3s |
| Block | Data unit | Default 128MB (or 256MB). Replicated across nodes |
| Replication | Fault tolerance | Default 3: replica 1 same node (local), replica 2 different rack, replica 3 same rack, different node |

## 💻 HDFS Key Commands

File Operations:

```bash
hdfs dfs -ls /path
hdfs dfs -put local.txt /hdfs/
hdfs dfs -get /hdfs/file.txt local.txt
hdfs dfs -rm /hdfs/file.txt
hdfs dfs -mkdir /new/path
```

Admin Commands:

```bash
hdfs dfsadmin -report
hdfs dfsadmin -safemode enter/leave
hdfs namenode -format
hdfs fsck /path
hdfs balancer
```

## ⚙️ MapReduce Job Structure

WordCount driver:

```java
public class WordCountDriver {
    public static void main(String[] args) throws Exception {
        Job job = Job.getInstance(new Configuration());
        job.setJarByClass(WordCountDriver.class);
        job.setJobName("WordCount");

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        job.setCombinerClass(WordCountReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        job.setNumReduceTasks(10);
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

Mapper:

```java
public static class WordCountMapper
        extends Mapper<LongWritable, Text, Text, IntWritable> {

    private final IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {
        String[] words = value.toString().split(" ");
        for (String w : words) {
            word.set(w);
            context.write(word, one);
        }
    }
}
```

Reducer:

```java
public static class WordCountReducer
        extends Reducer<Text, IntWritable, Text, IntWritable> {

    public void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```

> Tip: Use a combiner when the operation is associative (sum, count, max, min). The combiner class can be the same as the reducer class.

## 📦 Writable Data Types

| Class | Java Type | Use Case |
|---|---|---|
| BooleanWritable | boolean | True/false values |
| IntWritable | int | 32-bit integers |
| LongWritable | long | 64-bit integers, offsets |
| FloatWritable | float | 32-bit floats |
| DoubleWritable | double | 64-bit floats |
| Text | String (UTF-8) | Strings, keys |
| BytesWritable | byte[] | Binary data |
| MapWritable | Map | Key-value pairs |
| ArrayWritable | Array | Homogeneous arrays |

Custom Writable example:

```java
public class Employee implements Writable {
    public Text name;
    public IntWritable age;
    public DoubleWritable salary;

    public void write(DataOutput out) throws IOException {
        name.write(out);
        age.write(out);
        salary.write(out);
    }

    public void readFields(DataInput in) throws IOException {
        name.readFields(in);
        age.readFields(in);
        salary.readFields(in);
    }
}
```

> Critical: `write()` and `readFields()` order MUST match exactly! This is a common source of data corruption.

## 🧵 YARN Configuration

| Parameter | Default | Description | File |
|---|---|---|---|
| yarn.nodemanager.resource.memory-mb | 8192 | Memory available for containers (MB) | yarn-site.xml |
| yarn.nodemanager.resource.cpu-vcores | 8 | CPU cores available | yarn-site.xml |
| yarn.scheduler.minimum-allocation-mb | 1024 | Min container memory | yarn-site.xml |
| yarn.scheduler.maximum-allocation-mb | 8192 | Max container memory | yarn-site.xml |
| mapreduce.job.reduces | 1 | Number of reduce tasks | mapred-site.xml |
| mapreduce.reduce.memory.mb | 1024 | Reducer container memory | mapred-site.xml |
| mapreduce.map.java.opts | -Xmx1024m | Mapper JVM heap size | mapred-site.xml |

Key Memory Configuration formula: `-Xmx` (JVM heap) = ~75% of container memory

```text
Container: 2 GB  -> -Xmx1536m
Container: 4 GB  -> -Xmx3072m
Container: 8 GB  -> -Xmx6144m
```

## 🎛️ MapReduce Configuration Parameters

| Parameter | Default | Tuning Guidance |
|---|---|---|
| mapreduce.task.io.sort.mb | 100 | Increase if mappers produce large output. Keep < 50% of mapper memory |
| mapreduce.map.sort.spill.percent | 0.80 | Spill when 80% full. Lower = more spills, more I/O |
| mapreduce.reduce.shuffle.parallelcopies | 5 | Increase to 10-15 for faster network (parallel fetch from mappers) |
| mapreduce.map.output.compress | false | Set to true, use Snappy for faster shuffles |
| mapreduce.map.output.compress.codec | DefaultCodec | Use `org.apache.hadoop.hive.ql.io.compress.SnappyCodec` |
| mapreduce.job.reduces | 1 | Set to (num_nodes x 1.5) for parallelism |

## ▶️ Essential MapReduce & YARN Commands

Job Submission:

```bash
yarn jar job.jar DriverClass input output
hadoop jar job.jar DriverClass input output
yarn application -list
yarn application -status APP_ID
```

Monitoring:

```bash
yarn logs -applicationId APP_ID
hadoop job -list
hadoop job -status JOB_ID
mapred job -counter JOB_ID "group" "counter"
```

Job Control:

```bash
yarn application -kill APP_ID
hadoop job -kill JOB_ID
yarn nodemanager
yarn resourcemanager
```

Debugging:

```bash
hadoop jar job.jar DriverClass -Dmapreduce.framework.name=local input output
hadoop fs -cat /path/file | head -20
hdfs dfs -text /path/sequence-file
hadoop classpath
```

## 📥 InputFormats & OutputFormats Quick Reference

| Class | Key Type | Value Type | Use Case |
|---|---|---|---|
| TextInputFormat | LongWritable (offset) | Text (line) | Default for text files |
| KeyValueTextInputFormat | Text (key) | Text (value) | Tab/comma separated key-value |
| SequenceFileInputFormat | Any Writable | Any Writable | Binary, fast, intermediate data |
| NLineInputFormat | LongWritable | Text | N lines per split |
| CombineFileInputFormat | LongWritable | Text | Multiple small files -> single split |

## 🧩 Design Patterns Quick Reference

| Pattern | Problem | Solution | Example |
|---|---|---|---|
| Top-K | Find top 100 items | Mapper: heap(K), Reducer: merge heaps | Top trending products |
| Secondary Sort | Pre-sort before reduce | Composite key + custom comparator | Sorted output per group |
| Reduce-Side Join | Join 2 large datasets | Tag + key, join in reducer | USER join ORDER |
| Broadcast Join | Join with small dataset | Load small dataset in mapper memory | Enriching with dimension table |
| In-Mapper Combine | Reduce shuffle data | HashMap per mapper, emit in cleanup() | Counting, aggregation |
| Deduplication | Remove duplicates at scale | Hash key, emit once per group | User dedupe |
| Filtering/Sampling | Subset large data | Mapper: emit with probability | Random sampling |
| Windowing | Process time-windowed batches | Mapper: extract hour bucket, reduce: compute | Hourly aggregates |

## 🚀 Performance Tuning Checklist

Map Phase Optimization:
- Reduce input split size (more parallelism)
- Add input filtering (skip unnecessary data)
- Optimize mapper logic (avoid objects, cache)
- Use local variables instead of class fields
- Check input data size (is it really 100TB?)

Shuffle Phase Optimization:
- Enable map output compression (Snappy)
- Add combiner (if associative/commutative)
- Increase parallel copy threads (5 -> 15)
- Increase sort buffer (100MB -> 200MB)
- Check for data skew (hot keys?)

Reduce Phase Optimization:
- Increase number of reducers (1 -> 100+)
- Check for hotspot reducers
- Use custom partitioner for skew
- Optimize reducer logic
- Stream process instead of buffering

System-Wide Optimization:
- Reserve adequate memory per container
- Set correct JVM heap (-Xmx)
- Enable compression (final output)
- Monitor GC overhead
- Check cluster health (node failures?)

## 🔧 Troubleshooting Quick Guide

| Symptom | Cause | Fix |
|---|---|---|
| Task OOMKilled | Unbounded memory growth | Increase container, emit periodically, use streaming |
| One reducer much slower | Data skew, hot key | Custom partitioner, secondary key salting |
| Shuffle very slow | Large mapper output, no compression | Enable Snappy compression, add combiner |
| NullPointerException | Null value, version mismatch | Add null checks, verify all nodes have same code |
| Job very slow (suddenly) | Data size changed, node failures | Check cluster health, data characteristics |
| Task timeout | Mapper/reducer too slow per record | Increase timeout, optimize code, or split work |
| Output wrong | Writable serialization mismatch | Check write/readFields order matches |
| Few mappers, low utilization | Large input splits | Decrease split size, use CombineFileInputFormat |

## 🗜️ Compression Codecs Reference

| Codec | Compression Ratio | Speed | Splittable | Best For |
|---|---|---|---|---|
| Gzip | Very High (10x) | Slow | No | Final output, archival (compress but not read in parallel) |
| Snappy | Medium (4x) | Fast | No | Map output, shuffle (best for intermediate data) |
| LZ4 | Low-Medium (3x) | Very Fast | No | Speed critical, low latency needed |
| Bzip2 | Highest (11x) | Very Slow | Yes | Final output, one-time processing |
| Deflate | High (8x) | Medium | No | Default, balanced |

Enable compression in code:

```java
job.setMapOutputCompressorClass(SnappyCodec.class);
conf.setBoolean("mapreduce.map.output.compress", true);
conf.set("mapreduce.map.output.compress.codec",
    "org.apache.hadoop.hive.ql.io.compress.SnappyCodec");
```

## 🔢 Counters

```java
enum COUNTERS {
    TOTAL, VALID, INVALID, FILTERED, SKIPPED
}

context.getCounter(COUNTERS.TOTAL).increment(1);
context.getCounter(COUNTERS.VALID).increment(1);

// In the driver, after the job completes:
Job job = Job.getInstance();
job.waitForCompletion(true);
Counters counters = job.getCounters();

long total = counters.findCounter(COUNTERS.TOTAL).getValue();
long valid = counters.findCounter(COUNTERS.VALID).getValue();
```

> Best Practice: Always use counters at decision points. They reveal data flow issues without examining actual data.

## ⚙️ Key Configuration Parameters by File

`core-site.xml`:

```xml
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://namenode:8020</value>
  <!-- Default file system (HDFS) -->
</property>
<property>
  <name>io.compression.codecs</name>
  <value><!-- comma-separated list of available codecs on cluster --></value>
</property>
```

`hdfs-site.xml`:

```xml
<property>
  <name>dfs.blocksize</name>
  <value>134217728</value> <!-- 128m; use 268435456 for 256m -->
</property>
<property>
  <name>dfs.replication</name>
  <value>3</value> <!-- Default replica count -->
</property>
<property>
  <name>dfs.namenode.name.dir</name>
  <value>/var/hadoop/name</value> <!-- NameNode metadata directory -->
</property>
<property>
  <name>dfs.datanode.data.dir</name>
  <value>/var/hadoop/data</value> <!-- DataNode block storage -->
</property>
```

`yarn-site.xml`:

```xml
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>resourcemanager-host</value> <!-- RM host address -->
</property>
<property>
  <name>yarn.nodemanager.resource.memory-mb</name>
  <value>8192</value> <!-- Total memory available per node -->
</property>
<property>
  <name>yarn.scheduler.minimum-allocation-mb</name>
  <value>1024</value> <!-- Minimum container size -->
</property>
```

`mapred-site.xml`:

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value> <!-- Use YARN (not local) -->
</property>
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>0.0.0.0:10020</value> <!-- Job history server -->
</property>
<property>
  <name>mapreduce.map.memory.mb</name>
  <value>1024</value> <!-- Mapper container memory -->
</property>
```

## ❌ Common Mistakes & How to Avoid

**Mistake: Single Reducer**
Setting `job.setNumReduceTasks(1)` for global aggregation.
Fix: Use partial aggregation. Each reducer aggregates its portion, final job aggregates partials.

**Mistake: Unbounded Memory**
Collecting all values in a HashMap, no bounds checking.
Fix: Emit every N records, clear the HashMap, or use a streaming approach.

**Mistake: No Validation**
Processing data without checking format, types, nulls.
Fix: Validate in the first mapper, use counters, emit invalid records separately.

**Mistake: Ignoring Locality**
Not considering where data is stored.
Fix: Let InputFormat handle this, verify mapper placement in logs.

## 🎯 Interview Quick Reference

Know these cold:
- HDFS Replication: Replica 1: same node (local), Replica 2: different rack, Replica 3: same rack, different node
- MapReduce Flow: Input -> Map -> Sort/Partition -> Shuffle -> Reduce -> Output
- Combiner: Works when operation is associative/commutative (sum, count, max, min, NOT average)
- Reducer Count: (nodes x 1.5) is a good starting point. Avoid 1 (hotspot), avoid 1000+ (too many files)
- Data Skew: One reducer gets 90% of data. Use a custom partitioner, secondary key salting
- OOMKill: Task exceeds container memory. Set -Xmx to 75% of container, emit periodically
- Bottleneck: Usually shuffle (50-70% of time). Optimize with compression, combiner, partitioning
- When NOT to use MapReduce: Real-time (<1s latency), interactive queries, very complex logic (use Spark)

Design questions approach:
- Ask clarifying questions first: data size, latency, cost constraints?
- Start simple, then optimize
- Discuss trade-offs: latency vs throughput vs cost
- Show awareness of pitfalls: skew, memory, network
- Consider operational aspects: monitoring, debugging, maintenance

## ✅ Pre-Job Submission Checklist

Code Quality:
- [ ] Unit tests for business logic (not Hadoop framework)
- [ ] Clean code, good variable names
- [ ] No hardcoded paths (use Configuration)
- [ ] Comprehensive logging (INFO level)
- [ ] Use counters at decision points

Configuration:
- [ ] Set mapper/reducer memory appropriately
- [ ] Set reducer count (not default 1)
- [ ] Enable compression on map output
- [ ] Add combiner if applicable
- [ ] Set input/output formats correctly

Testing:
- [ ] Test locally with small data
- [ ] Test on cluster with sample data
- [ ] Check output is correct (sample records)
- [ ] Verify counters make sense
- [ ] Check for null pointer, serialization issues

Monitoring:
- [ ] Monitor job progress (should see tasks complete)
- [ ] Check shuffle time (is it reasonable?)
- [ ] Monitor reducer load (any hotspots?)
- [ ] Check output file count (not too many)
- [ ] Set up alerts for job failure

## 📐 Quick Formulas & Rules of Thumb

Sizing:
- Block size: 128MB or 256MB
- Number of mappers: Total_Input_Size / Block_size
- Number of reducers: Num_Nodes x 1.5
- Container size: multiple of 512MB (e.g. 1, 2, 4, 8 GB)
- Heap size: 75% of container memory

Performance:
- Expected runtime: (Input_Size / Throughput)
- Typical throughput: 1-10 GB/min per node
- Shuffle is 50-70% of job time
- Combiner can reduce shuffle 5-20x
- Compression (Snappy): ~40-50% reduction

Storage:
- 3x replication = 3x storage cost
- Gzip: 10x compression (slow)
- Snappy: 4x compression (fast)
- Text vs Parquet: 5-10x difference
- Network: ~1 GB/s between racks

Scaling:
- Node capacity: 12-16 cores, 64-256 GB RAM
- Cluster: 10-1000+ nodes
- NameNode memory: ~150 bytes per file
- DataNode disk: 80-90% utilization
- Network: 10Gbps between racks typical

---

*Source: adapted from the Apache Hadoop cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
