Freshworks data engineer questions 

Here’s a **structured list of commonly asked Data Engineer interview questions at Freshworks**, based on typical hiring patterns for product-based SaaS companies.

Freshworks focuses heavily on:

* Data pipelines
* Spark / SQL
* Distributed systems
* ETL design
* Real-time + batch processing
* Problem solving + system thinking

---

# 🔹 1️⃣ SQL (Very Important – Expect Deep Questions)

### Core SQL

* Difference between `ROW_NUMBER`, `RANK`, `DENSE_RANK`
* Find 2nd highest salary
* Deduplicate records using SQL
* Window functions vs GROUP BY
* Write query for:

  * Rolling average
  * Cumulative sum
  * Cohort analysis
* Handling skew in joins
* Query optimization techniques

### Advanced

* Explain execution plan
* Index vs partition
* How to optimize slow query?
* Star vs Snowflake schema

---

# 🔹 2️⃣ Spark / Big Data

In Apache Spark:

* Spark architecture
* Driver vs Executor
* What happens when you run a Spark job?
* Narrow vs wide transformations
* Partition vs bucketing
* Repartition vs coalesce
* How to handle data skew?
* Broadcast join – when & why?
* Fault tolerance in Spark
* Small file problem
* How Spark handles memory?

### Coding Round

* Word count variation
* Log processing
* Aggregation problem using DataFrame API

---

# 🔹 3️⃣ System Design (Data Engineering)

Very common in product companies.

Example questions:

* Design a data pipeline for:

  * Real-time event ingestion
  * Clickstream processing
  * Analytics dashboard
* Design scalable ETL architecture
* Batch vs Streaming pipeline
* How to handle late arriving data?
* How to ensure data quality?
* Idempotent pipeline design
* How to handle schema evolution?

Expect discussion around:

* Kafka
* Spark
* Airflow
* Data Lake
* Warehouse
* Monitoring

---

# 🔹 4️⃣ Python / Coding

* List vs tuple difference
* Generator vs iterator
* Decorators
* Write LRU cache
* JSON flattening
* Memory optimization
* Pandas vs Spark

DSA medium-level problems:

* Sliding window
* Hashmap problems
* String manipulation
* Two pointers

---

# 🔹 5️⃣ Data Modeling

* Fact vs Dimension table
* SCD Types (Type 1, 2, 3)
* OLTP vs OLAP
* How to model subscription SaaS data?
* Handling multi-tenant data

---

# 🔹 6️⃣ Real-World Scenario Questions

These are common at Freshworks:

* A job runs slow suddenly — how do you debug?
* Data pipeline failed at 2 AM — what steps?
* Dashboard numbers are inconsistent — how to investigate?
* How to reduce cost of Spark job?
* How to detect duplicate events?

---

# 🔹 7️⃣ Behavioral (Product Company Focus)

* Explain a pipeline you built end-to-end
* How did you scale it?
* How did you optimize performance?
* Example of production incident you handled
* Conflict with analyst or backend team
* Handling ambiguity

---

# 🔥 Interview Pattern (Typical)

1. Online coding test (SQL + Python)
2. Technical round (Spark + SQL)
3. System design round
4. Hiring manager / cultural round

---

# 🎯 How to Prepare Specifically for Freshworks

Focus on:

* Writing optimized SQL fast
* Spark internals (not just API)
* Data pipeline design thinking
* Production debugging mindset
* Cost optimization awareness

Product companies care about:

> Ownership + Scalability + Performance thinking

---

If you'd like, I can:

* Give you **mock interview questions**
* Share a **sample system design answer**
* Or create a **1-week preparation plan** for Freshworks Data Engineer interview

Tell me your experience level (0–2 yrs / 3–5 yrs / 5+ yrs).
