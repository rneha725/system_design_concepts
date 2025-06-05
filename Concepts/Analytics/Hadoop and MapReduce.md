# 📘 **Hadoop System Design Notes (HDFS + MapReduce)**

---

## 🔷 1. What is Hadoop?

**Apache Hadoop** is a framework that allows distributed processing of large datasets using:

* **HDFS** (Hadoop Distributed File System) for **storage**
* **MapReduce** for **computation**
* Designed for **scale-out architecture** using commodity hardware

---

## 📂 2. HDFS (Hadoop Distributed File System)

### 🧱 Architecture

* **Master-slave model**

  * **NameNode (Master)**: Manages metadata – file names, directories, block locations, permissions.
  * **DataNodes (Slaves)**: Store actual data blocks.

### 📦 Storage Model

* Files are split into **fixed-size blocks** (default: `128MB`)
* Each block is **replicated** (default: 3 copies) across different nodes

### ✅ Characteristics

* **Write-once, read-many**
* Optimized for **sequential access**
* High throughput, not low-latency

---

## 🔧 3. Failure Handling in HDFS

HDFS is **fault-tolerant by design**. Here are the types of failures it handles:

### 💥 A. **DataNode Failure**

* **Technique Used**: **Block Replication**
* Each block is replicated to multiple DataNodes.
* If a DataNode fails:

  * Detected via **heartbeat** (sent every 3s to NameNode)
  * NameNode triggers **re-replication** of under-replicated blocks
  * No data loss as long as one replica survives

### 🧠 B. **NameNode Failure**

* **Technique Used**:

  1. **Secondary NameNode**: Takes periodic snapshots of metadata (FSImage + edit logs)

     * **Myth**: It is NOT a backup
  2. **HA NameNode (High Availability)**:

     * One **Active NameNode**, one **Standby**
     * Use **Zookeeper** for failover and coordination
     * **Shared Journal** (via NFS or Quorum Journal Manager) keeps metadata synced

### 🧱 C. **Block Corruption**

* HDFS maintains **checksums** for each block
* On read, checksum is verified:

  * If failed, HDFS fetches another replica
  * Corrupt copy is replaced

---

## 🔀 4. **MapReduce**

### 🧩 What is MapReduce?

* A **programming model** and **processing engine** for large-scale data.
* Based on two phases:

  1. **Map**: Converts input data to intermediate `(key, value)` pairs
  2. **Reduce**: Aggregates intermediate results by key

### 🧪 Example:

> Count the number of times each word appears in a file.

* **Map**: (“hello world”) → {hello:1, world:1}
* **Shuffle & Sort**: Groups by key
* **Reduce**: Sum the values → {hello: 10, world: 5}

### 🔁 Execution Flow:

1. Input split across multiple nodes
2. Mappers process data **where it's stored** (data locality)
3. Shuffling groups and moves data to reducers
4. Output stored in HDFS

### 🧯 Failure Handling in MapReduce:

* **TaskTracker failures**: Detected via heartbeats to the **JobTracker** or **YARN ResourceManager**
* **Re-run failed tasks** on a different node
* **Speculative execution**: Runs duplicate tasks in parallel and uses the faster result

---

## 🧰 5. Techniques Used Internally

| Technique                      | Purpose                                     |
| ------------------------------ | ------------------------------------------- |
| **Replication**                | Ensure data is available even if nodes fail |
| **Heartbeats**                 | Monitor node liveness                       |
| **Checkpointing**              | Store consistent FS metadata snapshots      |
| **Zookeeper**                  | High Availability, leader election          |
| **Checksums**                  | Detect and recover from data corruption     |
| **Data Locality Optimization** | Run compute close to where data is stored   |
| **Speculative Execution**      | Handle slow-running tasks                   |
| **Quorum Journal Manager**     | Metadata sync in HA setups                  |

---

## 🎯 6. When To Use Hadoop?

✅ Ideal Use Cases:

* Massive **batch processing**
* Log processing
* Offline ETL pipelines
* Clickstream analysis
* Data warehouse backend
* Machine Learning (training large models offline)

🚫 Not Ideal For:

* Low-latency queries
* Real-time analytics
* Small file processing
* Random read/writes (not write-friendly)

---

## ⚖️ 7. Tradeoffs

| Factor              | Pros                                      | Cons                                            |
| ------------------- | ----------------------------------------- | ----------------------------------------------- |
| **Performance**     | Good throughput via parallelism           | High latency, not interactive                   |
| **Fault Tolerance** | Replication + task re-execution           | More disk + network usage                       |
| **Cost**            | Commodity hardware friendly               | Still needs monitoring, tuning                  |
| **Flexibility**     | Scalable with pluggable tools (Hive, Pig) | Limited for real-time or update-heavy workloads |
| **Scalability**     | Can handle petabytes of data              | NameNode memory bottleneck for metadata         |

---

## 🚀 8. Example Applications

| Application         | Description                                 |
| ------------------- | ------------------------------------------- |
| **Facebook**        | Data warehouse on Hive + Hadoop             |
| **LinkedIn**        | Log processing, ETL pipelines               |
| **Twitter**         | Batch analysis of tweet history             |
| **Yahoo**           | First big adopter of Hadoop, custom tools   |
| **Retail Chains**   | Analyzing buying patterns from POS data     |
| **Financial Firms** | Risk modeling and historical data crunching |

---

## 🧠 9. Alternatives & Comparisons

| Tech                     | HDFS vs / MapReduce vs                      |
| ------------------------ | ------------------------------------------- |
| **S3**                   | Object store; more flexible, cloud-native   |
| **Spark**                | Faster (in-memory); better for iterative ML |
| **Delta Lake**           | Handles ACID on big data; supports updates  |
| **Kafka**                | For real-time data ingestion                |
| **Snowflake / BigQuery** | Fully managed modern cloud data warehouses  |

---

## 📝 Summary

| Feature             | Hadoop + HDFS                                |
| ------------------- | -------------------------------------------- |
| **Storage**         | Distributed file system (HDFS)               |
| **Compute**         | Batch (MapReduce)                            |
| **Fault Tolerance** | Replication, retries                         |
| **Data Size**       | Handles petabytes                            |
| **Scalability**     | Horizontal                                   |
| **Use Case**        | Batch processing at scale                    |
| **Limitation**      | Latency, no real-time, poor with small files |
