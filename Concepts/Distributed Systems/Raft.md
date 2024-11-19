
**Raft** is a consensus algorithm designed to manage a replicated state machine in a distributed system. It ensures that multiple servers agree on a sequence of actions, even in the presence of failures. Raft is commonly used in distributed systems to maintain consistency across nodes.

Here's an overview of how Raft works:

---

### **Key Concepts in Raft**
1. **State of Nodes**:
   - **Leader**: The central node that manages log replication.
   - **Follower**: Passively accepts requests from the leader.
   - **Candidate**: A node seeking election to become the leader.

2. **Terms**:
   - A term is a logical time unit in Raft, used to avoid conflicts. Each term begins with a leader election.

3. **Logs**:
   - A sequence of commands that represent changes to the system state.

---

### **Phases of Raft**
1. **Leader Election**:
   - When the current leader fails or is unreachable, a follower can become a candidate.
   - The candidate requests votes from other nodes. If it receives a majority of votes, it becomes the leader.
   - Ties or multiple candidates may result in repeated elections.

2. **Log Replication**:
   - The leader receives commands from clients and appends them to its log.
   - It sends `AppendEntries` requests to followers to replicate the log entries.
   - Followers append the entries to their logs and respond to the leader.
   - Once a log entry is replicated on a majority of nodes, the leader marks it as committed.

3. **Safety**:
   - Raft ensures that logs are consistent across nodes through the use of terms and indices.
   - A node only votes for a candidate with an up-to-date log.

---

### **How Raft Handles Failures**
1. **Leader Failure**:
   - A follower detects that it hasn’t heard from the leader (via heartbeats) within a timeout period.
   - It triggers a new election by becoming a candidate.
   - Elections resolve quickly in normal conditions because of randomized timeouts.

2. **Network Partitions**:
   - Only the partition with a majority can elect a leader.
   - When partitions heal, nodes synchronize logs to ensure consistency.

3. **Log Inconsistencies**:
   - Raft uses indices and terms to identify and resolve inconsistencies.
   - If a follower’s log is inconsistent, the leader overwrites it with the correct entries.

---

### **Core Properties of Raft**
1. **Safety**:
   - Only one leader can exist in a given term.
   - Committed entries are durable and will not be lost.

2. **Liveness**:
   - Raft guarantees progress as long as a majority of nodes are operational.

3. **Understandability**:
   - Designed to be simpler than similar algorithms like Paxos, making it easier to implement and debug.

---

Raft is widely used in distributed systems like **etcd**, **Consul**, and **CockroachDB** for managing consensus. Its combination of leader-based approach and simplicity makes it a practical choice for real-world systems.
