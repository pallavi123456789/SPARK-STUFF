Great question — and this is exactly the type of situation where people usually wonder:

**“Why do we need Kafka in the middle when StreamSets can directly send data to Databricks?”**

Let me explain this using a **real manufacturing factory test-data use case**, and show you **exactly why Kafka is inserted between StreamSets pipelines**, especially for *real‑time* industrial scenarios.

***

# ✅ **Why Kafka in the Middle? (Simple Answer)**

Because **factory test data is extremely high‑volume, bursty, real‑time, and mission‑critical**, and Kafka solves problems that StreamSets → Databricks alone *cannot* handle.

Kafka provides:

### **1. Buffering / decoupling** between the factory source and Databricks

### **2. Replayability** → you can reprocess historical events

### **3. High‑throughput ingestion** for spikes

### **4. Multiple consumers** → analytics, monitoring, ML, audit

### **5. Guaranteed event ordering and durability**

Databricks *cannot* directly absorb unpredictable factory data bursts.  
StreamSets *can fail* or slow down under sudden high loads.

Kafka absorbs everything.

***

# ⭐ **Real-Time Factory Test Data Use Case (End-to-End)**

Let’s say you have a **manufacturing plant** producing devices — phones, automotive ECUs, sensors, etc.

Each device goes through **factory test stations**:

*   Functional tests
*   Voltage, current tests
*   Thermal tests
*   Quality inspection
*   Calibration test results

Each test event produces real-time data like:

```json
{
  "device_id": "ABC123",
  "serial_number": "SN987654",
  "test_station": "THERMAL",
  "temperature": 84.3,
  "test_status": "PASS",
  "timestamp": "2026-01-14T10:33:35Z"
}
```

Devices are tested **hundreds or thousands per minute** → extremely high volume.

***

# 🧩 **Pipeline Breakdown**

## 🔹 1. **Source → StreamSets (Ingestion)**

StreamSets collector connects to factory equipment through:

*   OPC-UA
*   MQTT
*   FTP drops
*   Custom APIs
*   CSV log files
*   Edge database
*   PLC controllers

This cleans & normalizes raw machine data.

### StreamSets does:

*   Flattening machine data
*   Removing machine noise
*   Timestamp normalization
*   Enrichment (lookup device info)
*   Error routing
*   Quality validation

***

## 🔹 2. **StreamSets → Kafka (Real-time Event Bus)**

This is the *critical part*.

### **Why factory test data MUST go through Kafka:**

### **① Data Spikes (bursty traffic)**

*   A batch of devices arrives → *data spike*
*   Machines restart → *burst of events*
*   Complex test runs → *spiky output*

Kafka absorbs spikes with *zero data loss*.

### **② Decouple the factory from Databricks**

If Databricks or StreamSets Consumer pipeline goes down:

*   **Kafka retains data for hours/days**
*   No testing line ever stops
*   No data is lost
*   Replay from offset when systems recover

This is *mission-critical* in manufacturing.

### **③ Replayability**

If you need:

*   To rebuild ML features for defect prediction
*   To fill data gaps
*   To reprocess historic test events
*   To recompute KPIs

Kafka allows replaying from **any offset**.

### **④ Multiple consumers**

Kafka feeds multiple downstream systems *simultaneously*:

1.  Live dashboard (Grafana/Databricks SQL)
2.  ML pipelines (Databricks features)
3.  Quality alerts engine
4.  Data lake storage
5.  Audits & compliance logs

You can't do this if you go StreamSets → Databricks directly.

***

## 🔹 3. **Kafka → StreamSets (Consumption)**

This pipeline reads Kafka events and prepares them for Databricks ingestion.

This step performs:

*   Schema enforcement
*   Validation
*   Splitting into different tables
*   Error handling
*   Routing pass/fail data
*   Transforming JSON/AVRO into Delta-ready structure

StreamSets simplifies handling bad records, corrupt messages, schema changes, etc.

***

## 🔹 4. **StreamSets → Databricks (Gold Layer / ML / Analytics)**

Final data lands into:

### **Bronze** (raw)

### **Silver** (clean)

### **Gold** (aggregated KPIs)\*\*

Databricks uses:

*   **Delta Live Tables**
*   **Auto Loader**
*   **MLflow**
*   **Feature Store**
*   **Power BI dashboards / Databricks SQL dashboards**

Usecases:

*   Predict which devices may fail QC
*   Identify faulty test stations
*   Real-time Quality Index (FTY, RTY)
*   Trend analysis

***

# 📌 **Concrete Real-Time Use Case Example**

A phone factory produces **20,000 devices per hour**.

Each device goes through 8 test stations.

→ **160,000 test events per hour (\~45 events/sec average, \~200–500/sec bursty)**

### Without Kafka

*   StreamSets → Databricks pipeline slows down
*   Factory test stations start backpressure → FILE DROPS
*   Data loss during network/cluster outages
*   No replay → **lost test results = lost production traceability**

### With Kafka

*   All events are buffered
*   StreamSets can consume at safe steady rate
*   If Databricks stops for 30 minutes → no impact
*   No data loss
*   Complete auditability
*   Real-time dashboards always up to date
*   Historical reprocessing possible

***

# 🏆 **Summary: Why Kafka is absolutely necessary**

| Need                              | Why Kafka solves it                            |
| --------------------------------- | ---------------------------------------------- |
| Bursty factory data               | Kafka buffers and handles high-velocity spikes |
| Zero data loss                    | Replication + durable storage                  |
| Replayability                     | Re-run entire history for ML/Audit             |
| Decoupling                        | Source & target operate independently          |
| Multiple consumers                | Dashboards, ML, archival, alerts               |
| Fault tolerance                   | Pipeline outages don’t affect production       |
| Scaling to millions of events/day | Kafka is built exactly for this                |

***
Got it—sounds like your round mixed hands-on data wrangling (Pandas), an API ingestion task, and some systems/experience questions (Spark optimization + pipeline design + behavioral).

If you want, I can turn this into a quick practice kit for you. In the meantime, here’s a compact prep guide with examples and checklists you can reuse.

***

## 1) Pandas Transformations (5 typical tasks + patterns)

Below are common transformation types with concise, interview-ready Pandas examples and their PySpark equivalents so you can switch if needed.

### a) Filtering + Conditional Columns

```python
import pandas as pd
df = pd.DataFrame({
    "user_id":[1,2,3,4],
    "country":["IN","US","IN","DE"],
    "spend":[120,50,200,0]
})

# filter
df_in = df[df["country"] == "IN"]

# conditional column
df["segment"] = pd.cut(
    df["spend"],
    bins=[-1,0,100,200,10**9],
    labels=["zero","low","mid","high"]
)
```

**PySpark equivalent**

```python
from pyspark.sql import functions as F
df_in = df.filter(F.col("country")=="IN")
df = df.withColumn(
    "segment",
    F.when(F.col("spend")==0,"zero")
     .when((F.col("spend")>0) & (F.col("spend")<=100),"low")
     .when((F.col("spend")>100) & (F.col("spend")<=200),"mid")
     .otherwise("high")
)
```

### b) GroupBy Aggregations

```python
agg_df = df.groupby("country", as_index=False).agg(
    total_spend=("spend","sum"),
    users=("user_id","nunique")
)
```

**PySpark**

```python
agg_df = df.groupBy("country").agg(
    F.sum("spend").alias("total_spend"),
    F.countDistinct("user_id").alias("users")
)
```

### c) Joins + Handling Duplicates

```python
orders = pd.DataFrame({"user_id":[1,1,2,3], "order_id":[10,11,20,30]})
merged = df.merge(orders, on="user_id", how="left").drop_duplicates()
```

**PySpark**

```python
merged = df.join(orders, on="user_id", how="left").dropDuplicates()
```

### d) Window Functions (rank, rolling)

```python
# Rank by spend within country
df["rank_in_country"] = (
    df.sort_values(["country","spend"], ascending=[True,False])
      .groupby("country")["spend"]
      .rank(method="dense", ascending=False)
)

# Rolling 3-day sum (assuming a date index)
# s.rolling('3D').sum() for time-based; or fixed window rolling(3)
```

**PySpark**

```python
from pyspark.sql.window import Window
w = Window.partitionBy("country").orderBy(F.col("spend").desc())
df = df.withColumn("rank_in_country", F.dense_rank().over(w))
```

### e) Reshaping (pivot/unpivot)

```python
sales = pd.DataFrame({
    "country":["IN","IN","US","US"],
    "month":["2025-01","2025-02","2025-01","2025-02"],
    "revenue":[100,150,200,80]
})
pivoted = sales.pivot_table(index="country", columns="month", values="revenue", aggfunc="sum").reset_index()
```

**PySpark**

```python
pivoted = (sales.groupBy("country").pivot("month").sum("revenue"))
```

**Tips:**

*   Prefer vectorized operations (avoid `apply` in loops).
*   Keep dtypes clean (`astype`, `to_datetime`) to avoid silent bugs.
*   For joins—verify keys, cardinality, and row-count deltas.

***

## 2) API Task (3 questions): Reading + Auth + Pagination/Errors

Here’s a robust Python pattern you can adapt quickly in interviews.

### Basic GET with headers and params

```python
import os, time, requests

BASE_URL = "https://api.example.com/v1/items"
API_KEY = os.getenv("API_KEY")  # or given directly in prompt
headers = {"Authorization": f"Bearer {API_KEY}", "Accept": "application/json"}

def fetch_page(page: int, page_size: int = 100):
    params = {"page": page, "page_size": page_size}
    r = requests.get(BASE_URL, headers=headers, params=params, timeout=30)
    if r.status_code == 429:  # rate limited
        time.sleep(int(r.headers.get("Retry-After", "2")))
        return fetch_page(page, page_size)
    r.raise_for_status()
    return r.json()

# pagination (page-based)
items = []
page = 1
while True:
    data = fetch_page(page)
    batch = data.get("items", data)  # depending on API shape
    if not batch:
        break
    items.extend(batch)
    if len(batch) < 100:  # last page
        break
    page += 1
```

### Auth patterns you might be asked about

*   **Bearer token** (above).
*   **API key in header or query**: `{"x-api-key": "..."}`
*   **OAuth2 Client Credentials** (short-lived tokens, refresh token logic).
*   **HMAC signatures** (compute hash on timestamp + path + secret).

### Resilience

*   Retries with **exponential backoff** for 5xx/timeout.
*   Defensive JSON parsing and schema version checks.
*   Idempotent writes downstream (upserts with natural key).
*   Logging request IDs (if provided via headers like `x-request-id`).

### Loading into Pandas

```python
import pandas as pd
df_api = pd.json_normalize(items)
```

***

## 3) Spark Optimization (talking points + examples)

Even if you wrote Pandas in the pad, these points often come up:

**Core strategies**

*   **Minimize shuffles**: co-locate joins by **repartition** on join key; avoid wide transformations unless necessary.
*   **Broadcast small tables**: `broadcast(dim)` for hash joins when dim << 10–100 MB (cluster-dependent).
*   **Cache/persist** hot intermediate DataFrames\*\* only\*\* when reused; unpersist later.
*   **Predicate pushdown** and **column pruning**: use Parquet/Delta; select only needed columns early.
*   **Skew handling**: salting keys, `skewHint`, or adaptive query execution (AQE).
*   **Partitioning strategy**: write data partitioned by low-cardinality, high-selectivity columns (e.g., date, region).
*   **File sizing**: coalesce to target \~128–512 MB per file to avoid small-file problem.
*   **Delta Lake optimizations**: `OPTIMIZE`, `ZORDER` on frequently filtered columns (if available).
*   **UDF caution**: prefer built-ins; if needed, use Pandas UDFs (vectorized) over standard Python UDFs.

**Snippet (broadcast + partition-aware join)**

```python
from pyspark.sql import functions as F
from pyspark.sql import SparkSession
from pyspark.sql.functions import broadcast

# Repartition large fact by join key; broadcast small dim
fact = fact_df.repartition("user_id")
joined = fact.join(broadcast(dim_df), on="user_id", how="left")
```

**Explain & validation**

```python
joined.explain()  # check for BroadcastHashJoin, shuffle stages
```

***

## 4) Pipeline Design (system design talking points)

Structure your answer across **ingest → store → transform → serve → observe**.

*   **Ingestion**:
    *   Sources: batch files, APIs, CDC (Debezium/Stream).
    *   Contracts: schema registry or OpenAPI; throttle control; retries & DLQ.
*   **Storage layers**:
    *   Bronze (raw, append-only) → Silver (cleaned, conformed) → Gold (serving/aggregates).
    *   Formats: Parquet/Delta for columnar + ACID (Delta/Iceberg).
*   **Transform**:
    *   Orchestration (Airflow/Databricks Jobs).
    *   Idempotent jobs (UPSERT/MERGE on natural keys).
    *   Data quality (Great Expectations/Deequ)—null checks, range checks, referential integrity.
*   **Serving**:
    *   Warehouse (Synapse/BigQuery/Snowflake) or feature store for ML.
    *   Index for point lookups (Elasticsearch) if needed.
*   **Observability**:
    *   Metrics (latency, throughput), data freshness SLAs, lineage, audit logs.
    *   Alerts on anomalies (missing partition, volume drop/spike).
*   **Non-functionals**:
    *   Cost governance (auto-stop, spot pools), security (PII tagging, KMS), access (RBAC/ABAC).

**MERGE/UPSERT (Delta example)**

```sql
MERGE INTO silver.users AS t
USING bronze.users_incr AS s
ON t.user_id = s.user_id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *
```

***

## 5) Behavioral / Past Experience (STAR quick frames)

**Example: Spark optimization**

*   **S**: “Large fact (2B rows) joined with 5M-row dim caused 2h runtime.”
*   **T**: “Reduce runtime <30 min.”
*   **A**: “Repartitioned on join key, broadcasted dim, pruned columns, cached hot DF, fixed skew with salting.”
*   **R**: “Runtime 2h → 22m; cost -35%; validation via data checks & `explain`.”

**Example: Pipeline reliability**

*   **S**: “API ingestion flaky (429/5xx), causing gaps.”
*   **A**: “Exponential backoff, circuit breaker, DLQ with replay, idempotent upserts, incremental bookmarks by `updated_at`.”
*   **R**: “99.5% SLO met; zero duplicate records.”

***

## 6) Quick Practice Pack (you can paste into CoderPad)

### Dataset seeds

```python
import pandas as pd
import numpy as np

np.random.seed(0)
n=1000
df = pd.DataFrame({
    "user_id": np.random.randint(1, 201, size=n),
    "country": np.random.choice(["IN","US","DE","GB"], size=n, p=[0.5,0.2,0.2,0.1]),
    "event_dt": pd.to_datetime("2025-02-01") + pd.to_timedelta(np.random.randint(0,30,size=n), unit="D"),
    "spend": np.random.gamma(2.0, 40.0, size=n).round(2)
})
orders = (df[["user_id","event_dt"]]
          .assign(order_id=lambda x: np.arange(len(x))+1000))
```

### 5 transformations to practice

1.  **Filter** to Feb-2025 India users and **rank** by spend within each day.
2.  Compute **7-day rolling** spend per user.
3.  **Join** orders and compute **repeat buyers** per country.
4.  **Pivot** daily revenue by country → columns are countries.
5.  Derive **RFM** buckets (Recency, Frequency, Monetary) and segment users.

If you want, I can generate the complete reference solutions.

***

## 7) Common Pitfalls Interviewers Check

*   Incorrect merge keys / unintentionally duplicating rows.
*   Using `apply`/row-wise loops instead of vectorized ops.
*   Not validating row counts before/after joins/filters.
*   Ignoring timezones and `to_datetime` parsing.
*   Missing retry/backoff for APIs; not considering rate limits.
*   Over-caching in Spark; not unpersisting; broadcasting huge tables.

***

## 8) What would help you most next?

I can:

*   Create a **timed mock round** (1 Pandas coding set + 1 API task + 5 design/behavioral questions).
*   Produce a **cheat sheet** (1–2 pages) you can skim before the round.
*   Build a **Jupyter notebook** with the practice dataset and solutions.

Tell me which you prefer, and any specifics the interviewer emphasized (e.g., pagination style, auth type, or Spark storage format).
Great—here are **concise, interview‑ready Pandas solutions** for each task.  
Assumptions:

*   `df` has columns: `user_id` (int), `country` (str), `event_dt` (datetime), `spend` (float).
*   `orders` has at least: `user_id`, `order_id` (unique per order), optionally `event_dt` of the order.
*   If you don’t already have `orders`, I included a tiny helper to derive it from `df` (commented).

```python
import pandas as pd
import numpy as np

# --- If you need a quick orders table from events (uncomment if needed) ---
# orders = (df[['user_id','event_dt']]
#           .sort_values(['user_id','event_dt'])
#           .assign(order_id=lambda x: np.arange(len(x)) + 1000))
```

***

## 1) Filter to Feb‑2025 India users & rank by spend within **each day**

```python
# Ensure datetime
df['event_dt'] = pd.to_datetime(df['event_dt'])

# Filter: India + Feb 2025
mask = (df['country'].eq('IN')) & (df['event_dt'].dt.to_period('M') == pd.Period('2025-02'))
df_feb_in = df.loc[mask].copy()

# Normalize to the day for per-day ranking
df_feb_in['day'] = df_feb_in['event_dt'].dt.normalize()

# Rank spend within each day (dense ranking; change to method='min' if preferred)
df_feb_in['rank_in_day'] = (
    df_feb_in.groupby('day')['spend'].rank(method='dense', ascending=False)
)

# Result: df_feb_in has rows only for India in Feb 2025 with rank per day
```

***

## 2) Compute **7‑day rolling** spend per user (time‑aware window)

```python
# Sort to ensure stable window calculation
df = df.sort_values(['user_id', 'event_dt'])

# Rolling 7 days per user (includes current day)
roll7 = (
    df.groupby('user_id', group_keys=False)
      .apply(lambda g: g.set_index('event_dt')['spend'].rolling('7D', min_periods=1).sum())
      .reset_index(level=0, drop=True)
)

df['spend_7d'] = roll7.values
```

> Note: This is a **time-based** rolling window (`'7D'`), not “last 7 rows”. It handles multiple events per day correctly.

***

## 3) Join orders and compute **repeat buyers per country**

```python
# Count orders per user (nunique to be safe)
order_counts = orders.groupby('user_id')['order_id'].nunique().rename('order_count')

# Stable user→country mapping (latest known country; adjust if you need "first" or a dimension table)
user_country = (
    df.sort_values(['user_id','event_dt'])
      .drop_duplicates('user_id', keep='last')[['user_id','country']]
)

# Join and flag repeat buyers (>=2 orders)
buyers = user_country.merge(order_counts, on='user_id', how='left').fillna({'order_count': 0})
buyers['is_repeat'] = buyers['order_count'] >= 2

# Aggregate per country
repeat_buyers_per_country = (
    buyers.groupby('country')['is_repeat']
          .sum()
          .reset_index(name='repeat_buyers')
)

# (Optional) also compute total buyers and repeat rate
buyers_per_country = buyers.groupby('country')['user_id'].nunique().reset_index(name='total_buyers')
repeat_metrics = repeat_buyers_per_country.merge(buyers_per_country, on='country')
repeat_metrics['repeat_rate'] = repeat_metrics['repeat_buyers'] / repeat_metrics['total_buyers']
```

***

## 4) Pivot **daily revenue by country** → columns are countries

```python
daily_rev = (
    df.assign(day=df['event_dt'].dt.normalize())
      .groupby(['day', 'country'], as_index=False)['spend'].sum()
      .rename(columns={'spend': 'revenue'})
)

daily_rev_pivot = (
    daily_rev.pivot(index='day', columns='country', values='revenue')
             .fillna(0)
             .sort_index()
)

# daily_rev_pivot: index = day, columns = countries, values = revenue
```

***

## 5) Derive **RFM** buckets (Recency, Frequency, Monetary) & segment users

```python
# Choose a snapshot date (e.g., next day after last event)
snapshot_date = df['event_dt'].max().normalize() + pd.Timedelta(days=1)

# Frequency: count distinct orders (preferred) or events if orders not available
if 'orders' in globals():
    freq = orders.groupby('user_id')['order_id'].nunique().rename('frequency')
else:
    freq = df.groupby('user_id').size().rename('frequency')  # fallback

rfm = (
    df.groupby('user_id').agg(
        last_purchase=('event_dt', 'max'),
        monetary=('spend', 'sum')
    )
    .join(freq, how='left')
)

# Recency in days (lower is better)
rfm['recency'] = (snapshot_date - rfm['last_purchase'].dt.normalize()).dt.days

# Create R/F/M quartile scores (1–4). Use rank to avoid qcut duplicate-bin issues.
rfm['R_score'] = pd.qcut(rfm['recency'].rank(method='first'), 4, labels=[4,3,2,1]).astype(int)  # recent => higher score
rfm['F_score'] = pd.qcut(rfm['frequency'].rank(method='first'), 4, labels=[1,2,3,4]).astype(int) # more freq => higher
rfm['M_score'] = pd.qcut(rfm['monetary'].rank(method='first'), 4, labels=[1,2,3,4]).astype(int)  # more spend => higher

rfm['RFM_score'] = rfm['R_score'] + rfm['F_score'] + rfm['M_score']

# Simple, explainable segmentation rules (tune to your needs)
def segment_row(r, r_th=3, f_th=3, m_th=3):
    if (r.R_score >= r_th) and (r.F_score >= f_th) and (r.M_score >= m_th):
        return 'Champions'
    if (r.R_score >= r_th) and (r.F_score >= f_th):
        return 'Loyal'
    if (r.R_score <= 2) and (r.M_score >= m_th):
        return 'At Risk'
    if (r.F_score >= f_th):
        return 'Potential Loyalist'
    return 'Hibernating'

rfm['segment'] = rfm.apply(segment_row, axis=1)

# rfm now contains: recency, frequency, monetary, scores, and segment per user
```

***

### Quick sanity checks (optional in interviews)

```python
# 1) After join/aggregations, verify row counts make sense
assert repeat_metrics['total_buyers'].sum() == buyers['user_id'].nunique()

# 2) Rolling window monotonicity (7D spend shouldn’t drop unless outside window)
df_check = df.sort_values(['user_id','event_dt'])
```

If you share your actual column names (and whether orders come separately or are inferred from events), I’ll tailor these snippets 1:1 to your dataset and return the final outputs you can paste into CoderPad.

Got it, Pallavi! Here are the **PySpark** equivalents for all five tasks—clean, interview‑ready, and using idiomatic DataFrame + Window APIs.

> **Assumptions**
>
> *   `df`: columns → `user_id` (int), `country` (string), `event_dt` (timestamp), `spend` (double).
> *   `orders`: columns → `user_id` (int), `order_id` (string/int), optionally `event_dt` (timestamp).
> *   Spark 3.x; imports shown below.

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Ensure types
df = df.withColumn("event_dt", F.to_timestamp("event_dt"))
```

***

## 1) Filter to **Feb‑2025 India users** & **rank** by spend **within each day**

```python
df_feb_in = (
    df
    .filter((F.col("country") == "IN") &
            (F.year("event_dt") == 2025) &
            (F.month("event_dt") == 2))
    .withColumn("day", F.to_date("event_dt"))
)

w_day = Window.partitionBy("day").orderBy(F.col("spend").desc())

df_feb_in_ranked = df_feb_in.withColumn("rank_in_day", F.dense_rank().over(w_day))
# -> columns: ..., day, rank_in_day
```

***

## 2) **7‑day rolling** spend per user (time‑aware)

Use a **range window** keyed by event time in **seconds** to make the frame “last 7 days including current row”.

```python
# Order key in seconds since epoch (monotonic for timestamps)
df_ts = df.withColumn("ts_seconds", F.col("event_dt").cast("long"))

w_7d = (
    Window
    .partitionBy("user_id")
    .orderBy(F.col("ts_seconds"))
    .rangeBetween(-7*24*60*60, 0)  # last 7 days (in seconds), inclusive
)

df_roll7 = df_ts.withColumn("spend_7d", F.sum("spend").over(w_7d)).drop("ts_seconds")
# -> each row has spend_7d, a rolling 7-day sum per user
```

> This handles **multiple events per day** correctly and does not just use “last 7 rows”.

***

## 3) Join `orders` and compute **repeat buyers per country**

```python
# 1) Order count per user (defensive: distinct order_id)
order_counts = orders.groupBy("user_id").agg(F.countDistinct("order_id").alias("order_count"))

# 2) Stable user→country mapping (latest country by event_dt)
w_latest = Window.partitionBy("user_id").orderBy(F.col("event_dt").desc())
user_country = (
    df.withColumn("rn", F.row_number().over(w_latest))
      .filter(F.col("rn") == 1)
      .select("user_id", "country")
)

# 3) Join + repeat flag (>=2 orders)
buyers = (
    user_country.join(order_counts, on="user_id", how="left")
                .fillna({"order_count": 0})
                .withColumn("is_repeat", (F.col("order_count") >= 2).cast("int"))
)

# 4) Aggregate per country
repeat_buyers_per_country = (
    buyers.groupBy("country")
          .agg(F.sum("is_repeat").alias("repeat_buyers"))
)

# (Optional) total buyers & repeat rate
buyers_per_country = buyers.groupBy("country").agg(F.countDistinct("user_id").alias("total_buyers"))

repeat_metrics = (
    repeat_buyers_per_country
    .join(buyers_per_country, on="country", how="inner")
    .withColumn("repeat_rate", F.col("repeat_buyers") / F.col("total_buyers"))
)
```

***

## 4) Pivot **daily revenue by country** → columns are countries

```python
daily_rev = (
    df.withColumn("day", F.to_date("event_dt"))
      .groupBy("day", "country")
      .agg(F.sum("spend").alias("revenue"))
)

daily_rev_pivot = (
    daily_rev.groupBy("day")
             .pivot("country")
             .sum("revenue")
             .fillna(0.0)
             .orderBy("day")
)
# -> index-like 'day' rows; one column per country; values = revenue
```

***

## 5) **RFM** buckets (Recency, Frequency, Monetary) & segmentation

```python
# Snapshot = next day after max event date (normalize to date)
snapshot_date = df.agg(F.max("event_dt").alias("max_dt")).select(
    F.date_add(F.to_date("max_dt"), 1).alias("snapshot")
).collect()[0]["snapshot"]

# Frequency: distinct orders per user (preferred). Fallback: count events.
freq = (
    orders.groupBy("user_id").agg(F.countDistinct("order_id").alias("frequency"))
    if "orders" in globals() else
    df.groupBy("user_id").agg(F.count("*").alias("frequency"))
)

# Monetary + last purchase date
mon_last = (
    df.groupBy("user_id")
      .agg(
          F.max("event_dt").alias("last_purchase"),
          F.sum("spend").alias("monetary")
      )
      .withColumn("recency", F.datediff(F.lit(snapshot_date), F.to_date("last_purchase")))
)

rfm_users = (
    mon_last.join(freq, on="user_id", how="left")
            .fillna({"frequency": 0})
            .select("user_id", "recency", "frequency", "monetary")
)

# Compute quartile thresholds using approxQuantile
qR = rfm_users.approxQuantile("recency",   [0.25, 0.50, 0.75], 0.01)
qF = rfm_users.approxQuantile("frequency", [0.25, 0.50, 0.75], 0.01)
qM = rfm_users.approxQuantile("monetary",  [0.25, 0.50, 0.75], 0.01)

# Scores: Recency (lower=better), Frequency/Monetary (higher=better)
rfm_scored = (
    rfm_users
    # R score (reverse: smaller recency => higher score)
    .withColumn("R_score",
        F.when(F.col("recency") <= F.lit(qR[0]), F.lit(4))
         .when(F.col("recency") <= F.lit(qR[1]), F.lit(3))
         .when(F.col("recency") <= F.lit(qR[2]), F.lit(2))
         .otherwise(F.lit(1)).cast("int")
    )
    # F score (larger => higher score)
    .withColumn("F_score",
        F.when(F.col("frequency") <= F.lit(qF[0]), F.lit(1))
         .when(F.col("frequency") <= F.lit(qF[1]), F.lit(2))
         .when(F.col("frequency") <= F.lit(qF[2]), F.lit(3))
         .otherwise(F.lit(4)).cast("int")
    )
    # M score (larger => higher score)
    .withColumn("M_score",
        F.when(F.col("monetary") <= F.lit(qM[0]), F.lit(1))
         .when(F.col("monetary") <= F.lit(qM[1]), F.lit(2))
         .when(F.col("monetary") <= F.lit(qM[2]), F.lit(3))
         .otherwise(F.lit(4)).cast("int")
    )
    .withColumn("RFM_score", F.col("R_score") + F.col("F_score") + F.col("M_score"))
)

# Simple, explainable segments
rfm_segmented = (
    rfm_scored
    .withColumn("segment",
        F.when((F.col("R_score") >= 3) & (F.col("F_score") >= 3) & (F.col("M_score") >= 3), F.lit("Champions"))
         .when((F.col("R_score") >= 3) & (F.col("F_score") >= 3), F.lit("Loyal"))
         .when((F.col("R_score") <= 2) & (F.col("M_score") >= 3), F.lit("At Risk"))
         .when((F.col("F_score") >= 3), F.lit("Potential Loyalist"))
         .otherwise(F.lit("Hibernating"))
    )
)
```

***

### Quick validation & tips (Spark)

*   **Window perf**: For big data, ensure you have proper partitioning (e.g., `df.repartition("user_id")` before wide window ops).
*   **Skew**: If a few `user_id`s have huge volumes, consider salting or AQE to mitigate skew during windows/joins.
*   **File formats**: Store inputs as Parquet/Delta for predicate pushdown and column pruning, especially for the daily pivot and RFM aggregates.

If you share your exact schemas (dtypes) or a small sample, I can plug these in and return a runnable notebook-style snippet tailored to your dataset.

