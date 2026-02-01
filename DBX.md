Absolutely! Let’s go full **Databricks-focused interview prep** — these are the questions you’ll see in **FAANG / Big Tech / Enterprise Databricks interviews**, covering **PySpark, performance, Delta Lake, SQL, ML pipelines, and platform features**. I’ll also give **answers and hints on what interviewers look for**.

---

# 🧠 DATABRICKS INTERVIEW QUESTIONS & ANSWERS

---

## **1️⃣ Platform & Architecture**

### Q1: What is Databricks and how is it different from vanilla Spark?

**Answer:**

> “Databricks is a managed platform for Apache Spark that provides optimized runtimes, auto-scaling clusters, collaborative notebooks, and built-in integrations with Delta Lake and ML workflows. Unlike vanilla Spark, it offers managed infrastructure, automatic version upgrades, and advanced performance optimizations (Photon engine, caching, AQE enhancements).”

**Follow-up tip:**
Interviewers may ask about **Photon, Auto-Scaling, and Databricks Runtime versions**.

---

### Q2: What is a Databricks cluster?

**Answer:**

> “A Databricks cluster is a set of virtual machines configured to run Spark jobs. It can be interactive (for notebooks) or job clusters (for production pipelines). Clusters can auto-scale and terminate idle nodes automatically.”

**FAANG tip:**
Know **driver vs worker nodes**, **auto-scaling**, and **job vs interactive clusters**.

---

### Q3: Explain Databricks workspace.

**Answer:**

> “A workspace is the collaborative environment where notebooks, libraries, dashboards, and jobs are managed. It supports collaboration via folders, permissions, and versioning.”

---

## **2️⃣ PySpark + Databricks**

### Q4: How do you optimize PySpark jobs in Databricks?

**Answer:**

* Enable **AQE**: `spark.sql.adaptive.enabled = True`
* Use **broadcast joins** for small tables
* **Salting / handling skew** for hot keys
* **Partition tuning**: `repartition()` / `coalesce()`
* **Delta caching** and **DBIO caching** (Databricks-specific)
* Avoid `groupByKey` → use `reduceByKey` or DataFrame aggregations
* Drop unused columns early

---

### Q5: How do you check job performance in Databricks?

**Answer:**

* **Spark UI**: accessible from cluster or job page
* **Ganglia / Spark Metrics** for memory/CPU usage
* **Query Profile in SQL Analytics** for Delta queries
* Key metrics: shuffle read/write, task durations, GC time, spill, skew

---

### Q6: How to handle skewed joins in Databricks?

**Answer:**

* Enable **AQE skew handling** (`spark.sql.adaptive.skewJoin.enabled = True`)
* **Salting hot keys manually**
* **Broadcast small table** if possible
* **Filter hot keys separately**

*(Same as advanced PySpark skew patterns.)*

---

## **3️⃣ Delta Lake**

### Q7: What is Delta Lake?

**Answer:**

> “Delta Lake is a storage layer on top of data lakes that adds ACID transactions, schema enforcement, versioning, and time travel. It ensures reliability for big data pipelines.”

---

### Q8: How do you perform **upserts** (merge) in Delta Lake?

```python
from delta.tables import DeltaTable

deltaTable = DeltaTable.forPath(spark, "/mnt/delta/users")
deltaTable.alias("tgt").merge(
    updates_df.alias("upd"),
    "tgt.user_id = upd.user_id"
).whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()
```

**FAANG tip:**

* Be ready to explain **when to use `merge` vs `update` + `insert`**
* Understand **performance considerations** for large tables

---

### Q9: How does Delta Lake handle schema evolution?

**Answer:**

* `mergeSchema = True` in `write` allows adding columns
* Enforces **schema validation** for ACID safety
* Supports **time travel** with schema changes

---

### Q10: How do you optimize Delta tables?

* **Z-Ordering**: clustering for efficient queries

```python
delta_table.optimize().where("date >= '2026-01-01'").zorderBy("customer_id")
```

* **VACUUM**: remove old files
* **Partitioning**: ensure queries filter partitions
* **Caching**: for frequently queried data

---

## **4️⃣ Databricks SQL / Analytics**

### Q11: What is Databricks SQL?

**Answer:**

> SQL engine in Databricks for analytics, BI dashboards, and ad-hoc queries over Delta tables. Supports **query profiles**, visualizations, and integration with BI tools like Tableau or PowerBI.

---

### Q12: How do you debug a slow SQL query in Databricks?

* Use **Query Profile** → shows:

  * Shuffle operations
  * Stage breakdown
  * Bottlenecks like skew or large scans
* Apply **partition pruning**, **Z-order**, or **caching**

---

## **5️⃣ Jobs & Pipelines**

### Q13: How do you schedule jobs in Databricks?

* Use **Jobs UI**: schedule notebooks or JARs/Python scripts
* Cron-based schedules or triggered workflows
* Can chain tasks with dependencies

---

### Q14: What are **Delta Live Tables (DLT)**?

**Answer:**

> Managed ETL pipelines with **declarative transformations**, auto-quality checks, and built-in monitoring. Handles incremental updates automatically.

---

### Q15: How do you monitor Databricks jobs?

* Job Runs page → task duration, cluster events
* Spark UI → stage/task-level performance
* Alerts / metrics integration with **MLflow** or external monitoring

---

## **6️⃣ ML / MLflow**

### Q16: How do you track experiments in Databricks?

* Use **MLflow** integration
* Logs parameters, metrics, and models
* Supports automated model versioning and deployment

---

### Q17: How do you deploy a model in Databricks?

* Register model in MLflow
* Deploy as REST API endpoint or batch scoring
* Integrate with Databricks Jobs or streaming pipelines

---

## **7️⃣ Advanced / Senior-Level Questions**

### Q18: Explain Photon engine

**Answer:**

> “Photon is Databricks’ vectorized query engine optimized for Delta Lake and SQL workloads. It accelerates query execution using SIMD instructions and optimized memory access.”

---

### Q19: Explain **Databricks Runtime versions**

* Standard runtime → Spark + Python/Scala
* ML runtime → adds ML libraries
* GPU runtime → for deep learning
* Photon → query acceleration
* **Interview tip:** know version differences

---

### Q20: How do you handle massive tables (>10B rows) efficiently?

* Partition intelligently
* Use **Delta + Z-order + caching**
* Broadcast small tables
* Enable **AQE + skew handling**
* Reuse intermediate results with **caching**

---

### 💡 Interview Tip:

> Most Databricks interviews mix **PySpark + Delta Lake + SQL + Jobs orchestration + performance tuning**. Be ready to explain **why you choose a specific join, caching strategy, or partitioning scheme**.

---

I can create the **next-level cheat sheet**:

**“Top 50 Databricks Interview Q&A with PySpark + Delta + UI clues + best answers”** – perfect for **FAANG prep**.

Do you want me to create that next?

---

# 🏆 TOP 50 DATABRICKS INTERVIEW Q&A

---

## **1️⃣ Databricks Platform Basics**

1. **Q:** What is Databricks?
   **A:** Managed Spark platform with collaborative notebooks, optimized runtime, auto-scaling, Delta Lake, and ML/BI integrations.

2. **Q:** Difference between Databricks and open-source Spark?
   **A:** Managed runtime, auto-scaling, Photon engine, built-in Delta Lake, UI, notebooks, cluster management.

3. **Q:** What is a Databricks cluster?
   **A:** Collection of VMs for running Spark jobs; driver + workers; supports auto-scaling and termination.

4. **Q:** Driver vs Worker node?
   **A:** Driver: runs SparkContext, coordinates jobs. Worker: executes tasks, stores RDD partitions.

5. **Q:** What is Databricks workspace?
   **A:** Collaborative environment for notebooks, dashboards, jobs, libraries, and access control.

---

## **2️⃣ PySpark / Spark API**

6. **Q:** What is a shuffle?
   **A:** Data redistribution across partitions (network + disk I/O) for joins, aggregations, or repartitioning.

7. **Q:** Which transformations cause shuffle?
   **A:** `groupByKey`, `reduceByKey`, `join`, `distinct`, `repartition`.

8. **Q:** Difference between `reduceByKey` and `groupByKey`?
   **A:** `reduceByKey` aggregates locally before shuffle → less data shuffled. `groupByKey` shuffles everything.

9. **Q:** How to optimize joins in PySpark?
   **A:** Broadcast small tables, pre-partition large tables, AQE skew handling, select/drop columns early.

10. **Q:** How to debug slow jobs in Spark UI?
    **A:** Look at slowest stage, shuffle read/write, task duration, GC time, disk spill, skewed partitions.

11. **Q:** How do you handle skewed joins?
    **A:** AQE skew join, salting hot keys, separate hot keys, broadcast small table.

12. **Q:** What’s the difference between `coalesce()` and `repartition()`?
    **A:** `coalesce()` reduces partitions without shuffle; `repartition()` reshuffles data for balanced partitions.

13. **Q:** Why does Spark spill to disk?
    **A:** Data exceeds executor memory during shuffle, aggregation, or caching.

14. **Q:** How to avoid disk spill?
    **A:** Increase partitions, cache carefully, reduce columns, use broadcast joins, and AQE.

15. **Q:** How to read task skew in Spark UI?
    **A:** Task duration chart → long tail indicates skew; shuffle read/write → huge partition.

---

## **3️⃣ Delta Lake**

16. **Q:** What is Delta Lake?
    **A:** ACID-compliant storage layer on data lakes with versioning, time travel, schema enforcement.

17. **Q:** How to perform UPSERT in Delta?

```python
deltaTable.alias("tgt").merge(
    updates_df.alias("upd"),
    "tgt.id = upd.id"
).whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()
```

18. **Q:** How to enable schema evolution in Delta?

```python
df.write.option("mergeSchema", True).mode("append").format("delta").save(path)
```

19. **Q:** What is Z-Ordering?
    **A:** Data clustering technique to co-locate similar values → improves query pruning.

20. **Q:** How to optimize Delta table performance?

* Partition by filter columns
* Z-order frequently queried columns
* VACUUM to remove old files
* Delta caching

21. **Q:** What is Delta Time Travel?
    **A:** Query previous versions using:

```python
spark.read.format("delta").option("versionAsOf", 5).load(path)
```

22. **Q:** What are VACUUM and OPTIMIZE?

* `VACUUM` → removes old files
* `OPTIMIZE` → compact small files, optionally Z-order

23. **Q:** How to handle small files problem?
    **A:** OPTIMIZE + Delta Auto Compaction + batching small writes

24. **Q:** How to do incremental load in Delta?
    **A:** Use merge or append with watermark/filter on timestamp column

25. **Q:** How to handle schema mismatch errors?

* Enable `mergeSchema`
* Use `select` to align columns

---

## **4️⃣ Joins + Skew Patterns**

26. **Q:** How to detect skew in join?
    **A:** Spark UI → task duration + shuffle read distribution → one/few huge partitions.

27. **Q:** AQE Skew Join configuration

```python
spark.conf.set("spark.sql.adaptive.enabled", True)
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", True)
```

28. **Q:** Manual salting pattern for hot keys

```python
from pyspark.sql.functions import floor, rand, explode, array, lit
```

* Add `salt` column to hot table
* Explode small table by salt
* Join on key + salt

29. **Q:** How to handle NULL hot keys?
    **A:** Filter or replace NULL keys → prevent skew

30. **Q:** Pre-partition tables for join

```python
df1 = df1.repartition("key")
df2 = df2.repartition("key")
```

31. **Q:** Broadcast join in PySpark

```python
from pyspark.sql.functions import broadcast
df1.join(broadcast(df2), "key")
```

32. **Q:** When to use sort-merge join?

* Large tables, broadcast not feasible
* Default Spark join

33. **Q:** How to fix skewed aggregation?

* Salting
* AQE adaptive aggregation
* Increase shuffle partitions

---

## **5️⃣ Spark SQL + Analytics**

34. **Q:** How to debug slow SQL query in Databricks?

* Query Profile → stage breakdown, shuffle, skew, spill
* Partition pruning
* Z-ordering
* Delta caching

35. **Q:** Difference between DataFrame API and SQL?

* DataFrame API → programmatic
* SQL → declarative, BI-friendly

36. **Q:** How to create Delta table in SQL?

```sql
CREATE TABLE users_delta
USING DELTA
PARTITIONED BY (country)
LOCATION '/mnt/delta/users'
```

37. **Q:** How to do time-travel query in SQL?

```sql
SELECT * FROM users_delta VERSION AS OF 5
```

38. **Q:** How to monitor query metrics in Databricks SQL?

* Query Profile → shuffle, task duration, scan size

---

## **6️⃣ Jobs, Pipelines, DLT**

39. **Q:** How to schedule Databricks jobs?

* Jobs UI → notebooks, scripts, JARs
* Cron or triggered schedules

40. **Q:** What are job clusters?

* Ephemeral clusters launched per job → auto-terminate

41. **Q:** What are interactive clusters?

* Used for notebooks / exploration, long-running

42. **Q:** What is Delta Live Table (DLT)?

> Declarative, managed ETL pipelines with quality checks and automatic incremental updates

43. **Q:** How to monitor Databricks jobs?

* Job Runs UI
* Task durations + cluster events
* Spark UI for detailed analysis

44. **Q:** How to handle failed jobs?

* Retries
* Alerts
* Logging

---

## **7️⃣ ML & MLflow**

45. **Q:** How to track ML experiments?

* MLflow → logs parameters, metrics, models

46. **Q:** How to deploy a model in Databricks?

* Register in MLflow → REST API or batch scoring

47. **Q:** How to monitor model metrics?

* MLflow → dashboards, custom metrics

48. **Q:** How to integrate ML pipelines with Delta?

* Feature tables stored as Delta
* Incremental updates with Delta merge

49. **Q:** How to manage large datasets for ML?

* Partition & cache
* Delta tables for feature storage
* AQE for training data join

50. **Q:** How to handle skew in ML feature joins?

* Same as data joins: AQE, salting, broadcast, pre-partition

---

# 🔥 SPARK UI CLUES (INTERVIEW HACK)

| Symptom           | What it means       | Fix                                    |
| ----------------- | ------------------- | -------------------------------------- |
| One long task     | Skew / hot key      | Salting / AQE                          |
| High shuffle read | Join / aggregation  | Broadcast / reduce columns             |
| Disk spill        | Memory pressure     | Increase partitions / avoid groupByKey |
| SortMergeJoin     | Large join          | Broadcast if possible                  |
| Many small tasks  | Too many partitions | Coalesce / optimize                    |

---

# 💡 INTERVIEW TIP

* Always **map Spark UI stages → DataFrame / SQL transformations**
* Know **AQE + skew + broadcast + caching** patterns
* Explain **Delta Lake features** clearly
* Give **quantitative improvements** (time reduced, spill reduced)

---

Absolutely! Let’s dive into **PySpark Streaming + Kafka** — this is a hot topic in **FAANG and enterprise interviews**. I’ll structure it into **conceptual + code + UI/performance + real-world scenarios**, with **interview-ready answers**.

---

# 🧠 PYSPARK STREAMING + KAFKA INTERVIEW Q&A

---

## **1️⃣ Streaming Basics**

### Q1: What is PySpark Streaming?

**Answer:**

> “PySpark Streaming allows processing of live data streams using Spark’s structured streaming API. It supports micro-batch or continuous processing with fault tolerance and exactly-once semantics.”

---

### Q2: Difference between DStream and Structured Streaming?

| Feature         | DStream       | Structured Streaming   |
| --------------- | ------------- | ---------------------- |
| API             | RDD-based     | DataFrame / Dataset    |
| Fault-tolerance | Checkpointing | WAL + checkpointing    |
| Ease of use     | Lower         | Higher                 |
| Integration     | Limited       | Kafka, Delta, ML, JDBC |

> FAANG tip: **Structured Streaming is preferred today.**

---

### Q3: What is a micro-batch in Spark Streaming?

**Answer:**

> “Spark divides the incoming data stream into small batches (e.g., every 1 second) and processes them like mini batch jobs.”

---

### Q4: What is watermarking?

**Answer:**

> “Watermarking helps handle late-arriving data by defining a threshold for event time processing. It allows Spark to drop old state and prevent memory leaks.”

Example:

```python
df.withWatermark("event_time", "10 minutes")
  .groupBy("user_id", window("event_time", "5 minutes"))
  .count()
```

---

### Q5: What is the difference between append, update, and complete output modes?

| Mode     | Meaning           | When to use                                   |
| -------- | ----------------- | --------------------------------------------- |
| append   | Only new rows     | Stateless aggregations                        |
| update   | Only updated rows | Aggregations with keys                        |
| complete | Full output       | Full state aggregation (slow for large state) |

---

## **2️⃣ Kafka Integration**

### Q6: How to read a Kafka stream in PySpark?

```python
df = (spark.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "broker1:9092,broker2:9092")
      .option("subscribe", "topic_name")
      .option("startingOffsets", "earliest")
      .load())
```

* `startingOffsets` = earliest / latest / JSON
* Value comes as binary → decode with `.cast("string")`

---

### Q7: How to write back to Kafka?

```python
df.selectExpr("key", "value") \
  .writeStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "broker1:9092") \
  .option("topic", "output_topic") \
  .option("checkpointLocation", "/mnt/checkpoints/topic") \
  .start()
```

---

### Q8: How to ensure exactly-once processing with Kafka?

**Answer:**

* Enable **checkpointing**
* Use **write-ahead logs (WAL)**
* Use Kafka’s **idempotent producer** for writes

---

### Q9: What is the difference between Kafka consumer groups and Spark Streaming?

**Answer:**

* Kafka consumer group → multiple consumers for a topic partition
* Spark → parallelism determined by number of partitions + executors
* Spark structured streaming manages offsets internally → can commit offsets automatically

---

### Q10: How to handle late events from Kafka?

* Use **watermarking**
* Drop or aggregate late events
* Maintain **state with timeout**

```python
.withWatermark("event_time", "10 minutes")
.groupBy("user_id", window("event_time", "5 minutes"))
.count()
```

---

## **3️⃣ Stateful Processing**

### Q11: What is stateful streaming in PySpark?

* Keeps track of state across micro-batches
* Example: running totals per key

```python
df.groupBy("user_id").agg(sum("amount")).writeStream...
```

---

### Q12: How does Spark manage state?

* In memory + WAL checkpoint
* Expire old keys with watermark or timeout

---

### Q13: Difference between mapWithState and updateStateByKey?

| Feature              | mapWithState | updateStateByKey |
| -------------------- | ------------ | ---------------- |
| API                  | DStream      | DStream          |
| Memory               | Efficient    | High memory      |
| Structured Streaming | N/A          | N/A              |

> Today, use **Structured Streaming + flatMapGroupsWithState** for advanced stateful streaming.

---

## **4️⃣ Windowed Aggregations**

### Q14: How to do sliding windows in PySpark streaming?

```python
from pyspark.sql.functions import window

df.groupBy(window("event_time", "5 minutes", "1 minute"), "user_id") \
  .count()
```

* 5 min window, 1 min slide → overlapping windows

---

### Q15: How to deal with skew in streaming aggregation?

* Repartition by key
* Use **mapGroupsWithState** → avoid huge hot keys
* Drop or isolate skewed keys

---

## **5️⃣ Checkpointing and Fault Tolerance**

### Q16: Why is checkpointing important in structured streaming?

* Recovery from failure
* Maintain offsets for Kafka
* Maintain state for stateful operations

---

### Q17: Where to store checkpoint files?

* Durable storage: DBFS, S3, ADLS, HDFS

---

### Q18: Can you change checkpoint location mid-stream?

* No → must restart with new location
* Old checkpoints tie to application and query id

---

## **6️⃣ Performance Optimization**

### Q19: How to optimize micro-batch streaming in PySpark?

* Reduce batch size → lower latency
* Increase parallelism → repartition large streams
* Cache intermediate results
* Avoid wide transformations (joins/aggregations) on huge datasets

---

### Q20: How to optimize Kafka reads in Spark?

* Use **Kafka partitions = Spark partitions**
* Use **startingOffsets = latest** in production
* Tune **maxOffsetsPerTrigger** to control load

---

### Q21: How to monitor streaming jobs in Spark UI?

* Check **Streaming tab** → input rate, processing time, backlog
* Check **Stages** → long shuffle tasks indicate skew
* Check **Tasks** → GC, disk spill

---

### Q22: What is backpressure in Spark Streaming?

* When processing < input rate → backlog grows
* Enable `spark.streaming.backpressure.enabled = True`
* Dynamically reduces rate per partition

---

## **7️⃣ Real-World / Advanced Scenarios**

### Q23: How to handle exactly-once join with Kafka streams and batch table?

* Use **Structured Streaming + Delta table**
* Use **merge with idempotency**
* Enable **checkpointing**

---

### Q24: How to do multi-topic streaming in Kafka?

```python
df = spark.readStream.format("kafka") \
    .option("subscribePattern", "topic_.*").load()
```

---

### Q25: How to handle schema evolution in streaming Kafka + Delta?

* Use Delta `mergeSchema=True`
* Use `from_json` with evolving schema

---

### Q26: How to handle late-arriving events in windowed aggregations?

* Use **watermarks** + `allowedLateness`
* Maintain state until watermark threshold

---

### Q27: How to debug streaming latency?

* Streaming tab: inputRows vs processedRows
* Look for batch processing time > trigger interval → bottleneck
* Check GC / disk spill

---

### Q28: How to handle failures in streaming pipelines?

* Enable **checkpointing**
* Set **write-ahead log** (WAL) for sinks like Kafka
* Idempotent writes

---

### Q29: How to ensure end-to-end exactly-once semantics with Kafka + Delta Lake?

* Use **Structured Streaming + Delta Sink**
* Delta automatically handles idempotent updates with checkpointing

---

### Q30: How to scale streaming jobs in Databricks?

* Increase **executors / cores**
* Tune **trigger interval**
* Repartition input stream to match number of partitions
* Avoid single hot key in aggregation

---

# 🔥 FAANG INTERVIEW TIP – Streaming + Kafka

* Always **map Spark UI metrics → pipeline performance**
* Be ready to **explain watermarking, checkpointing, exactly-once semantics**
* Show **you know PySpark structured streaming code + Kafka integration**
* Bonus: mention **Delta Lake streaming sink** for idempotent writes

---
