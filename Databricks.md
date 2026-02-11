# Databricks Medallion Architecture

The **Medallion Architecture** is a data design pattern used in **Databricks Lakehouse** to progressively improve data quality and structure through layers:

> 🥉 **Bronze → 🥈 Silver → 🥇 Gold**

It helps organize data pipelines for scalability, governance, and analytics.

---

## 🥉 Bronze Layer (Raw Data)

![Image](https://media.licdn.com/dms/image/v2/D4D12AQFX8rkTEgQK1w/article-cover_image-shrink_600_2000/B4DZaHr0_sGgAY-/0/1746033137826?e=2147483647\&t=abU3u55oQ9_pVvCrQspIiD9boRvv_Je0UCsnnsujyBI\&v=beta)

![Image](https://www.altexsoft.com/static/blog-post/2024/4/984d355c-0793-4051-9c61-d8237412fdc6.jpg)

![Image](https://www.databricks.com/sites/default/files/inline-images/db-265-blog-img-1.png)

![Image](https://miro.medium.com/max/1002/1%2A6LtBtGVZdc5RMk44cK_gZw.png)

### 🔹 Purpose:

Store **raw, unprocessed data** exactly as received from source systems.

### 🔹 Characteristics:

* Append-only
* Stores original format (JSON, CSV, Parquet, etc.)
* Includes metadata (ingestion timestamp, source file name)
* Minimal transformation

### 🔹 Example:

* Raw IoT sensor data
* Kafka stream data
* CRM exports
* Log files

### 🔹 Technologies:

* Delta Lake
* Auto Loader
* Structured Streaming

---

## 🥈 Silver Layer (Cleaned & Enriched Data)

![Image](https://media.licdn.com/dms/image/v2/D4D12AQFX8rkTEgQK1w/article-cover_image-shrink_600_2000/B4DZaHr0_sGgAY-/0/1746033137826?e=2147483647\&t=abU3u55oQ9_pVvCrQspIiD9boRvv_Je0UCsnnsujyBI\&v=beta)

![Image](https://www.researchgate.net/publication/385683308/figure/fig1/AS%3A11431281289418709%401731172383814/mage-showing-a-data-pipeline-that-includes-stages-such-as-data-collection-data-cleaning.jpg)

![Image](https://www.chaosgenius.io/blog/content/images/2024/07/Medallion-Architecture.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AJxBpkbdv59eV-D57)

### 🔹 Purpose:

Clean, validate, and transform Bronze data.

### 🔹 Characteristics:

* Deduplicated
* Schema enforced
* Data type corrections
* Joins across multiple sources
* Handles nulls & corrupt records

### 🔹 Example:

* Cleaned customer table
* Validated transaction table
* Joined user + purchase dataset

### 🔹 Transformations:

* Remove duplicates
* Filter invalid rows
* Standardize date formats
* Business rule validations

---

## 🥇 Gold Layer (Business-Ready Data)

![Image](https://media.licdn.com/dms/image/v2/D4D12AQFX8rkTEgQK1w/article-cover_image-shrink_600_2000/B4DZaHr0_sGgAY-/0/1746033137826?e=2147483647\&t=abU3u55oQ9_pVvCrQspIiD9boRvv_Je0UCsnnsujyBI\&v=beta)

![Image](https://docs.databricks.com/aws/en/assets/images/lakehouse-diagram-865ba5c041f60df99ff6bee1ebaad26d.png)

![Image](https://learn.microsoft.com/en-us/fabric/enterprise/powerbi/media/aggregations-automatic/auto-aggregations.png)

![Image](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2021/09/08/DBBLOG-1495-featureimage.png)

### 🔹 Purpose:

Optimized for **analytics and BI reporting**.

### 🔹 Characteristics:

* Aggregated data
* Business-level tables
* Star/Snowflake schemas
* Optimized for performance

### 🔹 Example:

* Daily revenue summary
* Customer lifetime value table
* Sales dashboard dataset

### 🔹 Used By:

* Power BI
* Tableau
* Machine Learning models
* Data analysts

---

# 🔄 End-to-End Flow

```
Source Systems
      ↓
  Bronze (Raw)
      ↓
  Silver (Cleaned)
      ↓
  Gold (Aggregated / Business Ready)
      ↓
  BI / ML / Reporting
```

---

# 🚀 Why Medallion Architecture?

| Benefit         | Explanation                |
| --------------- | -------------------------- |
| Data Quality    | Improves progressively     |
| Reproducibility | Raw data always preserved  |
| Governance      | Clear separation of stages |
| Performance     | Optimized gold tables      |
| Scalability     | Handles batch + streaming  |

---

# 🎯 Interview-Ready Explanation (Short Version)

If asked in an interview:

> “Medallion architecture in Databricks is a layered data design approach with Bronze (raw), Silver (cleaned and transformed), and Gold (business-aggregated) layers. It ensures data quality, governance, and scalability while enabling both streaming and batch processing in the Lakehouse using Delta Lake.”

Handling **corrupt records in Medallion Architecture (Databricks Lakehouse)** is a very common interview and real-world question.

The key principle is:

> ❗ Never lose raw data.
> ✅ Quarantine bad records.
> ✅ Fix them downstream.

Let’s break it layer by layer.

---

# 🥉 Bronze Layer – Capture Everything (Even Corrupt Data)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AG7WXO-9LnXhyCsXc__iqbw.png)

![Image](https://editor.analyticsvidhya.com/uploads/64236image-9.png)

![Image](https://www.chaosgenius.io/blog/content/images/2024/03/Databricks-Autoloader-1.png)

![Image](https://community.databricks.com/tmcxu86974/attachments/tmcxu86974/data-engineering/42628/1/Screenshot%202025-01-24%20at%2011.52.16.png)

### 🎯 Goal: Do NOT drop corrupt data.

### ✅ Best Practices:

### 1️⃣ Use `_corrupt_record` column

When reading JSON/CSV:

```python
df = spark.read \
    .option("mode", "PERMISSIVE") \
    .option("columnNameOfCorruptRecord", "_corrupt_record") \
    .json("path")
```

This:

* Loads valid rows normally
* Stores malformed rows in `_corrupt_record`

---

### 2️⃣ Store Raw Data As-Is

* Append-only Delta table
* Add metadata columns:

  * `ingestion_timestamp`
  * `source_file_name`
  * `batch_id`

Example:

```python
df.withColumn("ingestion_time", current_timestamp()) \
  .write.format("delta") \
  .mode("append") \
  .saveAsTable("bronze_table")
```

---

### 3️⃣ NEVER filter corrupt data in Bronze

Bronze = System of Record
Even bad data must exist here.

---

# 🥈 Silver Layer – Clean & Quarantine

![Image](https://www.databricks.com/sites/default/files/inline-images/building-data-pipelines-with-delta-lake-120823.png)

![Image](https://www.oreilly.com/api/v2/epubs/urn%3Aorm%3Abook%3A9781492053187/files/assets/bmlp_0401.png)

![Image](https://delta-io.github.io/delta-rs/how-delta-lake-works/contents-of-delta-table.png)

![Image](https://miro.medium.com/max/1400/1%2AaBMIgVjk-Ikluokru9kAkg.png)

### 🎯 Goal: Separate valid vs invalid data

---

## Step 1: Split Good & Bad Records

```python
silver_df = spark.table("bronze_table")

good_records = silver_df.filter("_corrupt_record IS NULL")
bad_records = silver_df.filter("_corrupt_record IS NOT NULL")
```

---

## Step 2: Store Bad Records in Quarantine Table

```python
bad_records.write \
    .format("delta") \
    .mode("append") \
    .saveAsTable("quarantine_table")
```

Now you:

* Track bad records
* Analyze patterns
* Fix and reprocess later

---

## Step 3: Apply Data Validation Rules

Example:

```python
clean_df = good_records.filter(
    (col("amount") > 0) &
    (col("customer_id").isNotNull())
)
```

Invalid business-rule records also go to quarantine.

---

# 🥇 Gold Layer – Business Safe Data

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AJxBpkbdv59eV-D57)

![Image](https://blog.bismart.com/hubfs/Data%20Warehouse%20Data%20Lake%20Data%20Lakehouse.jpg)

![Image](https://docs.genesys.com/images/thumb/4/40/Aggregationlevels90014.png/500px-Aggregationlevels90014.png)

![Image](https://www.oreilly.com/api/v2/epubs/urn%3Aorm%3Abook%3A9780471412540/files/9780471412540_aggregate_fact_tables_image01.png)

### 🎯 Only validated, trusted data goes to Gold.

* No corrupt records
* No schema violations
* Fully cleaned
* Aggregated for reporting

---

# 🔥 Advanced Production Techniques

## ✅ 1. Use Auto Loader Rescue Data Column

```python
.option("rescuedDataColumn", "_rescued_data")
```

This captures unexpected columns (schema evolution).

---

## ✅ 2. Use Delta Expectations (DLT)

In Delta Live Tables:

```sql
CREATE LIVE TABLE silver_table
CONSTRAINT valid_amount EXPECT (amount > 0)
```

Invalid rows:

* Can be dropped
* Or redirected automatically

---

## ✅ 3. Alerting & Monitoring

* Track count of corrupt records
* Create dashboards
* Set alerts if threshold exceeded

Example metric:

```python
bad_records.count()
```

---

# 📊 Real-World Flow

```
Source Data
   ↓
Bronze
   - Raw JSON
   - _corrupt_record column
   - Append only
   ↓
Silver
   - Remove corrupt
   - Apply validation
   - Store bad in quarantine
   ↓
Gold
   - Aggregated
   - Fully trusted
```

---

# 🎯 Interview-Ready Answer (Strong Version)

If asked:

> “How do you handle corrupt records in Medallion Architecture?”

Answer:

> “In Bronze, we ingest data in permissive mode and capture corrupt records using `_corrupt_record` or rescued data columns without dropping anything. In Silver, we separate valid and invalid records, store invalid ones in a quarantine Delta table, and apply data quality validations. Only fully validated data is promoted to Gold for business consumption.”

---

If you'd like, I can also explain:

* 🔥 Streaming corrupt record handling
* 💡 Schema evolution handling
* 🚀 Delta Live Tables expectations
* 🧠 Real production architecture example

Just tell me your experience level (entry/mid/senior).

