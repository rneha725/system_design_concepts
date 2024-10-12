<sub>I need to write about the storage too, for now this document only oulines the inbuilt CDC</sub>

In **CockroachDB**, **Change Data Capture (CDC)** is handled through its native support for streaming changes (such as inserts, updates, and deletes) from the database in real-time to downstream systems. CDC is useful for keeping external systems (e.g., data lakes, caches, analytics platforms) synchronized with CockroachDB.

Here’s an overview of how CockroachDB handles CDC, the architecture choices involved, and how failures are managed:

### **CockroachDB CDC Overview**
CockroachDB’s CDC mechanism allows users to subscribe to a stream of row-level changes in one or more tables. The changes are exported in a format compatible with systems like Kafka or webhook consumers. These changes can be captured incrementally (since the last read) or continuously as data changes in the database.

### **Architecture Choices in CockroachDB CDC**

1. **Distributed Architecture**:
   - CockroachDB, being a distributed SQL database, uses a multi-node, shared-nothing architecture, where each node is responsible for ranges of data.
   - CDC operates in this distributed setting, allowing multiple nodes to capture changes from their respective data partitions.
   - The **Raft protocol** (used by CockroachDB for distributed consensus and replication) ensures that CDC captures changes in a consistent, fault-tolerant manner. Writes are always committed to a majority of replicas in a range before CDC events are published.

2. **Change Feed**:
   - CDC in CockroachDB is built around a **change feed** mechanism. This feed operates at the table level and emits changes in an ordered manner.
   - CockroachDB’s architecture ensures that once a change is written to the database and committed, it can be captured and exported through the change feed.
   - The feed captures **row-level changes**, including metadata about the type of operation (insert, update, or delete) and can be used to replicate these changes to external systems.

3. **Granularity and Flexibility**:
   - CDC can be configured at different granularities (table or row-level) and supports various sinks such as Kafka, cloud storage, or webhooks.
   - It supports **schema changes** dynamically, meaning that CDC streams adjust automatically to schema alterations like column additions or deletions.

### **Failure Handling in CDC**

1. **Raft and Consensus-Based Replication**:
   - CockroachDB’s underlying Raft protocol ensures that data is consistently replicated across multiple nodes before it's considered committed. This means that any changes that are captured by the CDC have already been agreed upon by the majority of replicas in a Raft group.
   - If a **leader** (Raft group leader) node handling a range fails, Raft elects a new leader, ensuring continued availability of the data. This process is transparent to the CDC process, so as long as the write is committed, it will be part of the CDC stream.

2. **Resilience to Node Failures**:
   - In the event of a node failure, CockroachDB redistributes the affected data ranges across remaining nodes. Since CDC operates at the range level, this redistribution ensures that the change feed continues to capture changes from the newly reassigned replicas.
   - **Change feed state** is persisted, and when a failed node recovers, it can resume change capture without losing events. This is particularly important in ensuring **at-least-once delivery** semantics, ensuring that downstream consumers don’t miss any changes.

3. **Handling Network Partitions**:
   - Network partitions are handled by Raft’s consensus mechanism. In cases of partitioning, changes are only published by the CDC after achieving a majority consensus across nodes. This prevents inconsistent states from being captured by the CDC stream.
   - If nodes become temporarily unavailable due to partitioning, the CDC feed pauses and resumes once connectivity is restored and consensus is achieved.

4. **Backpressure and Flow Control**:
   - If downstream systems (such as Kafka or webhook consumers) experience delays or failures in processing the change feed, CockroachDB’s CDC includes mechanisms to handle **backpressure**.
   - This means that if the sink cannot keep up with the incoming change data, the feed adjusts to prevent overwhelming the downstream system. CockroachDB retains the CDC state so that changes are not lost, and once the sink is ready to process again, it will receive the buffered changes.

5. **At-Least-Once Semantics**:
   - CockroachDB CDC provides **at-least-once delivery guarantees**, meaning each change will be delivered to the downstream system at least once.
   - While this ensures durability, it may result in duplicate events being sent if a failure occurs. Downstream consumers need to be idempotent or handle deduplication to manage this.

### **Failure Recovery in CDC**:
   - **Checkpointing**: CockroachDB keeps track of the **high-water mark** of the CDC stream, which is essentially a timestamp representing the last change that was processed. This enables the system to recover from failures without replaying the entire change history.
   - If a failure occurs (e.g., node crashes or network disruptions), the change feed can resume from the last checkpoint, ensuring no data loss and minimizing redundant work.

### **CDC Sinks and Integration**:
- CockroachDB’s CDC system supports **Kafka**, **Google Cloud Storage**, **Webhooks**, and custom connectors as sinks for streaming the change data.
- It also allows users to configure **transactional boundaries**, so only committed changes are emitted, ensuring no partially completed transactions are published to the change feed.

### **Conclusion**:
CockroachDB's CDC leverages the underlying Raft consensus mechanism to ensure strong consistency guarantees and fault tolerance. The distributed nature of CockroachDB enables CDC to capture changes reliably, even in the event of node failures or network partitions. Failures are handled through consensus protocols, range redistribution, and persistent change feed state, making the CDC system resilient to various types of failures. At-least-once delivery semantics and checkpointing further ensure that downstream consumers can recover from disruptions without losing changes.
