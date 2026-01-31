Exploratory Data Analysis (EDA) in PySpark is slightly different than in Pandas because you are often dealing with datasets too large to fit into memory. The goal is to use Spark’s distributed engine to aggregate and summarize data before bringing smaller, manageable chunks to the driver for visualization.

Here is a breakdown of the EDA workflow in PySpark.

---

## 1. Structural Exploration

Before looking at the data, you need to understand the "bones" of your dataset.

* **`df.printSchema()`**: Checks data types and nullability.
* **`df.dtypes`**: Returns a list of (column, type) tuples for programmatic checks.
* **`df.columns`**: Quick list of feature names.
* **`df.limit(5).toPandas()`**: A popular trick to view data in a clean, formatted table (only for a small number of rows!).

---

## 2. Statistical Summaries

PySpark provides built-in methods to grab the "big picture" numbers.

### The `describe` and `summary` methods

While `describe()` gives you count, mean, stddev, min, and max, `summary()` goes further by including percentiles.

```python
# General statistics
df.describe().show()

# More detailed statistics including quartiles
df.select("age", "salary").summary("count", "min", "25%", "50%", "75%", "max").show()

```

---

## 3. Missing Value Analysis

In big data, "nulls" are often the most important thing to find. You can count nulls across all columns using a list comprehension:

```python
from pyspark.sql.functions import col, count, when

null_counts = df.select([count(when(col(c).isNull(), c)).alias(c) for c in df.columns])
null_counts.show()

```

---

## 4. Univariate Analysis (Value Distributions)

This helps you understand the spread and balance of individual features.

### Categorical Columns: Frequency Counts

```python
df.groupBy("category_column").count().orderBy("count", ascending=False).show()

```

### Numerical Columns: Histograms

Spark doesn't plot directly, but you can calculate the bins and counts on the cluster:

```python
# Calculate histogram data in Spark
hist_data = df.select("age").rdd.flatMap(lambda x: x).histogram(10)
# Returns (bin_edges, counts) which you can then pass to Matplotlib

```

---

## 5. Bivariate & Multivariate Analysis

Exploring how variables relate to one another.

### Correlation Matrix

PySpark’s `StatGroupedData` or `Correlation` from MLlib can calculate Pearson or Spearman correlations.

```python
from pyspark.ml.stat import Correlation
from pyspark.ml.feature import VectorAssembler

# Convert columns to a single vector column
assembler = VectorAssembler(inputCols=["age", "salary", "experience"], outputCol="features")
df_vector = assembler.transform(df).select("features")

# Calculate correlation matrix
matrix = Correlation.corr(df_vector, "features").collect()[0][0]
print(matrix.toArray())

```

### Cross-Tabulation (Contingency Tables)

To see the frequency distribution between two categorical variables:

```python
df.stat.crosstab("gender", "department").show()

```

---

## 6. Outlier Detection

A common EDA task is identifying values that fall outside the typical range. Using the Interquartile Range (IQR) method:

```python
# Calculate IQR
quantiles = df.approxQuantile("salary", [0.25, 0.75], 0.05)
IQR = quantiles[1] - quantiles[0]
lower_bound = quantiles[0] - (1.5 * IQR)
upper_bound = quantiles[1] + (1.5 * IQR)

# Filter Outliers
outliers = df.filter((col("salary") < lower_bound) | (col("salary") > upper_bound))

```

---

## 7. Sampling for Visualization

Once you have narrowed down your focus, you can sample the data to use standard Python libraries like **Seaborn** or **Matplotlib**.

```python
# Sample 10% of the data without replacement
df_sample = df.sample(False, 0.1, seed=42).toPandas()

import seaborn as sns
sns.boxplot(x='category', y='value', data=df_sample)

```



Interviewing for **Warner Bros. Discovery (WBD)** means you need to speak the language of **high-throughput media streaming** (HBO Max/Discovery+). In a petabyte-scale streaming environment, traditional EDA (like `df.show()`) will crash your driver or cost a fortune in compute.

For an interview, you should pivot from "Exploratory Data Analysis" to **"Exploratory Stream Processing."**

---

## 1. The Architectural Shift: "Shifting Left"

When data is in the petabyte range, you don't "explore" the raw data; you explore the **aggregates**.

* **Sampling is King:** You cannot process all PBs for a quick look. Use `sample()` or `sampleBy()` to create a statistically significant "Bronze" subset for exploration.
* **Structured Streaming + Watermarking:** Explain how you handle late-arriving data (e.g., a user pauses a movie in a tunnel and the "view" event hits the server 10 minutes later).
* **The Medallion Architecture:** Explain how you perform EDA at each stage:
* **Bronze:** Schema validation and "junk" detection.
* **Silver:** Deduplication (crucial for streaming) and normalization.
* **Gold:** Business KPIs (Churn, Watch Time).



---

## 2. Real-Time Distribution Analysis (Sketching)

You cannot sort petabytes of data to find the median. Mention **Approximate Algorithms** (Data Sketches).

* **HyperLogLog (HLL):** Used for "Approximate Distinct Counts." (e.g., *How many unique users watched 'The Last of Us' in the last hour?*)
* **ApproxQuantile:** Used to find the 90th percentile of buffering latency across millions of streams without a full sort.

```python
# PySpark Approximate Quantile for Latency EDA
quantiles = df.stat.approxQuantile("buffering_ms", [0.5, 0.9, 0.99], 0.01) 
# 0.01 is the relative error margin - critical for PB scale

```

---

## 3. Dealing with "Data Skew"

In WBD's case, a "blockbuster" release causes a massive spike in data for a specific `content_id`. This is **Data Skew**.

* **The Interview Answer:** "I use `salting` to break up heavy partitions."
* **EDA Step:** Check partition distribution to see if one executor is doing 90% of the work.

```python
from pyspark.sql.functions import spark_partition_id

# EDA to find Skew
df.withColumn("partition_id", spark_partition_id()) \
  .groupBy("partition_id").count().show()

```

---

## 4. Specific WBD Use-Case EDA

Prepare these specific metrics for your interviewers:

| EDA Category | What you are looking for | Why it matters to WBD |
| --- | --- | --- |
| **Bitrate Stability** | Variance in `bitrate_kbps` over time windows. | Quality of Service (QoS). |
| **Event Drift** | Schema changes in JSON payloads from different device types (Roku vs. iOS). | Ensures downstream ML models don't break. |
| **Sessionization** | Gaps between `heartbeat` events. | Calculating "True Watch Time" vs. "Idle Time." |

---

## 5. Visualizing at Scale

Explain that you don't use Matplotlib on the raw stream. Instead:

1. **Aggregate** in Spark (Windowing).
2. **Push** to a time-series DB (like Druid or ClickHouse).
3. **Visualize** in Grafana or Looker.

---

## 6. The "Gotcha" Interview Questions

* **Q: "How do you handle a late event from 2 hours ago?"**
* **A:** "I define a **Watermark**. If the watermark is 1 hour, anything older than that is dropped to maintain state-store performance."


* **Q: "How do you detect a sudden drop in stream quality globally?"**
* **A:** "I use a **Sliding Window** (e.g., 5-minute window, sliding every 1 minute) to calculate the average error rate and compare it against a Z-score baseline."


In a PB-scale environment like WBD, you aren't just "looking" at data; you are hunting for **bottlenecks, skew, and signal** across a distributed cluster.

Here is the implementation code for the high-level strategies we discussed.

---

## 1. Dealing with Skew (The "Blockbuster" Problem)

If *House of the Dragon* drops a new episode, that `content_id` will overwhelm a single executor. We use **Salting** to spread that load.

**EDA to Detect Skew:**

```python
from pyspark.sql import functions as F

# Check if one partition is significantly larger than others
df.withColumn("partition_id", F.spark_partition_id()) \
  .groupBy("partition_id") \
  .count() \
  .orderBy(F.desc("count")) \
  .show()

```

**Implementation (Salting):**

```python
# Add a random 'salt' to the join key to redistribute the data
salt_bins = 100
df_salted = df.withColumn("salt", (F.rand() * salt_bins).cast("int"))
df_salted = df_salted.withColumn("salted_key", F.concat(F.col("content_id"), F.lit("_"), F.col("salt")))

# Now the join happens across 100 partitions instead of 1

```

---

## 2. Approximate EDA (The "Speed over Precision" Strategy)

You cannot run `countDistinct()` on a Petabyte stream without the cluster "hanging." Use **HyperLogLog (HLL)** for unique users and **Approximate Quantiles** for latency.

```python
# 1. Approx Distinct Count (Unique Streamers)
# 0.05 represents the standard deviation/error margin
unique_viewers = df.select(F.approx_count_distinct("user_id", 0.05)).collect()[0][0]

# 2. Approx Quantiles (Buffering Latency)
# Finds the 50th, 95th, and 99th percentile of buffering time
quantiles = df.stat.approxQuantile("buffering_ms", [0.5, 0.95, 0.99], 0.01)

print(f"P95 Latency: {quantiles[1]}ms")

```

---

## 3. Sliding Windows (Real-time Signal EDA)

WBD needs to know *right now* if the error rate is spiking. This requires **Sliding Windows** with **Watermarking** to handle late data.

```python
# EDA: Track Error Rates in 10-minute windows, sliding every 5 minutes
stream_eda = df \
    .withWatermark("event_time", "2 hours") \
    .groupBy(
        F.window("event_time", "10 minutes", "5 minutes"),
        "device_type",
        "error_code"
    ).count()

# Filter for specific high-priority errors (e.g., Playback Failure)
playback_errors = stream_eda.filter(F.col("error_code") == "ERR_5001")

```

---

## 4. Column-Level Profiling (The `expr` Way)

For PB-scale data, you often need to check if your JSON payloads are drifting (missing fields).

```python
# Using expr for a SQL-style 'CASE' to profile stream quality
profile_df = df.select(
    F.count("*").alias("total_events"),
    F.expr("count(CASE WHEN bitrate_kbps < 1000 THEN 1 END)").alias("low_quality_count"),
    F.expr("count(CASE WHEN device_id IS NULL THEN 1 END)").alias("missing_device_id"),
    F.avg(F.col("load_time_sec")).alias("avg_load_time")
)

```

---

## 5. Sessionization (Calculating Watch Time)

In streaming, a user doesn't "log out"; they just stop sending heartbeats. You define a session by the gap between events.

```python
from pyspark.sql.window import Window

# Define a window partitioned by user
window_spec = Window.partitionBy("user_id").orderBy("event_timestamp")

# Calculate time difference between consecutive events
df_with_gap = df.withColumn("prev_time", F.lag("event_timestamp").over(window_spec))
df_with_gap = df_with_gap.withColumn("diff_seconds", 
    F.unix_timestamp("event_timestamp") - F.unix_timestamp("prev_time"))

# Identify new sessions (if gap > 30 minutes)
df_sessions = df_with_gap.withColumn("is_new_session", 
    F.when(F.col("diff_seconds") > 1800, 1).otherwise(0))

```

### Pro-Interview Tip for WBD:

If they ask about **Storage Formats**, mention that for EDA on PBs of data, you'd prefer **Z-Ordering** or **Liquid Clustering** (if using Delta Lake). This allows Spark to "skip" reading gigabytes of files that don't contain the specific `user_id` or `content_id` you are looking for.


This mock case study response is designed to showcase your ability to handle **incident response** and **root cause analysis (RCA)** at the scale of a company like Warner Bros. Discovery.

### The Scenario: "The 10% Drop"

**Interviewer:** *"We notice a 10% drop in average bitrate across HBO Max globally in the last 15 minutes. How do you use PySpark to identify the cause in a petabyte-scale stream?"*

---

## Phase 1: Rapid Triage (The "Where" and "Who")

The first goal is to determine if this is a **Global Infrastructure** issue or a **Segmented** issue (ISP, Device, or Content).

**The Code:**
I would run a multi-dimensional aggregation using a sliding window to isolate the variance.

```python
from pyspark.sql import functions as F

# Filter to the last 30 mins and aggregate by segments
triage_df = stream_df \
    .withWatermark("timestamp", "10 minutes") \
    .filter("timestamp > current_timestamp() - interval 30 minutes") \
    .groupBy("isp", "device_type", "region") \
    .agg(
        F.avg("bitrate_kbps").alias("avg_br"),
        F.count("session_id").alias("sample_size")
    ) \
    .withColumn("drop_severity", F.expr("avg_br / global_baseline_bitrate"))

# Look for segments where drop_severity < 0.90
triage_df.filter("drop_severity < 0.9").orderBy("drop_severity").show()

```

---

## Phase 2: Analyzing the "Long Tail" (Outlier Detection)

If the average is down, it might be because a specific "cluster" of users is experiencing a total outage. I'll use **Z-Scores** to find statistical outliers in the stream.

**The Code:**

```python
# Calculate Stats for the window
stats = stream_df.select(
    F.avg("bitrate_kbps").alias("mu"),
    F.stddev("bitrate_kbps").alias("sigma")
).collect()[0]

# Flag sessions that are 3 standard deviations below the mean
outliers_df = stream_df.withColumn("z_score", 
    (F.col("bitrate_kbps") - stats['mu']) / stats['sigma']
).filter("z_score < -3")

# EDA: Is there a common denominator among these outliers?
outliers_df.groupBy("cdn_provider", "content_id").count().show()

```

---

## Phase 3: Joining with Metadata (The "What")

WBD data is often "Snowflaked." The stream contains `content_id`, but the metadata (Is it 4K? Is it live?) sits in a Delta table. A **Broadcast Join** is essential here to prevent data skew and shuffle.

**The Code:**

```python
# Broadcast the small metadata table to all executors
from pyspark.sql.functions import broadcast

metadata_df = spark.table("content_metadata") # Info on 4K vs HD

# Join to see if the drop is only affecting 4K assets
analysis_df = stream_df.join(broadcast(metadata_df), "content_id")

analysis_df.groupBy("resolution_target").agg(F.avg("bitrate_kbps")).show()

```

---

## Phase 4: Final RCA & Reporting

Once the EDA identifies the culprit (e.g., *"CDN-A is throttling 4K traffic in the EMEA region"*), I would conclude the interview by explaining how to automate this:

1. **Metric Persistence:** Save the aggregated "Silver" data to a Delta table for long-term trend analysis.
2. **Alerting:** Hook the PySpark job into an Airflow/MWAA sensor or a PagerDuty API.
3. **Visualization:** Feed a sampled `toPandas()` subset into a dashboard for the NOC (Network Operations Center).

---

### Interview "Buzzwords" to Drop:

* **"Backpressure":** "I'd monitor if the Spark executors are struggling to keep up with the stream, causing the drop in reported metrics."
* **"Check-pointing":** "I'd ensure our structured stream has reliable check-pointing to recover from executor failures during the incident."
* **"Data Locality":** "I'd check if the processing is happening close to the data source to minimize latency during the investigation."

**Would you like me to provide a list of common "Gotcha" PySpark optimization questions that WBD often asks in senior roles?**
