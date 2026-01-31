To wrap up your prep for **Warner Bros. Discovery**, here is a hit-list of the "Gotcha" optimization questions. These are designed to see if you actually understand how Spark works under the hood or if you’ve just memorized syntax.

---

## 1. "How do you handle a Spark job that is failing with an OOM (Out of Memory) error on the Driver?"

**The Trap:** Most people suggest increasing `driver-memory`.
**The "Senior" Answer:** "While increasing memory is a quick fix, the root cause is usually a **large collect()** call or a **broadcast join** where the 'small' table isn't actually small. I would first check for `df.collect()` or `df.toPandas()` and replace them with writing to a sink or using `limit()`. If it’s a broadcast join, I’d check the size of the broadcasted table and potentially switch to a `SortMergeJoin`."

## 2. "What is the 'Small File Problem' and how do you solve it in a Streaming context?"

**The Context:** Streaming often writes many tiny files (one per trigger).
**The Solution:** * **Compaction:** Running a background job to `repartition(n)` and overwrite the directory.

* **Auto-optimize (Delta Lake):** Mentioning **Delta Lake's Auto-Optimize and Predictive Optimization** (which WBD likely uses) shows you know modern lakehouse architecture.

## 3. "Can you explain the difference between `repartition()` and `coalesce()`?"

**The "Senior" Answer:** * **`repartition()`**: Reshuffles all data. It’s expensive but creates equal-sized partitions. Use this when you need to **increase** partitions or fix **data skew**.

* **`coalesce()`**: Only **decreases** partitions by moving data to existing nodes (minimizing shuffle). It’s efficient but can result in "fat" partitions if not careful.

---

## 4. "How do you debug a 'Straggler' (one task taking 10x longer than others)?"

**The Strategy:** 1.  **Open Spark UI:** Look at the **Tasks** tab for that stage.
2.  **Identify Skew:** If the "Max" time is huge and the "Median" is small, you have **Data Skew**.
3.  **Action:** I would look for high-cardinality keys (like a single popular movie ID) and apply the **Salting** technique we discussed.

## 5. "What is the difference between Cache and Persist?"

**The Nuance:** * `cache()` is just a shortcut for `persist(MEMORY_AND_DISK)`.

* `persist()` allows you to define the **Storage Level** (e.g., `DISK_ONLY`, `MEMORY_ONLY_SER` for serialized data to save space).
* **Interview Tip:** Always mention `unpersist()`! It shows you care about cluster resource management.

---

## Final Interview Cheat Sheet: The "Scale" Keywords

When answering any question at WBD, try to weave these terms into your explanation:

| Term | Use Case |
| --- | --- |
| **Predicate Pushdown** | Filtering data at the source (Parquet/Delta) before it hits Spark. |
| **AQE (Adaptive Query Execution)** | Spark 3.x feature that re-optimizes query plans at runtime based on statistics. |
| **Z-Ordering** | Organizing data files by multiple dimensions (e.g., `user_id` and `timestamp`) for faster EDA. |
| **Stateful Streaming** | Maintaining user session info over time (vital for calculating "Watch Time"). |

---
