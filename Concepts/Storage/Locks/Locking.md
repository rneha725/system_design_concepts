### Optimistic Locking vs Pessimistic Locking

**Locking** in databases is used to control access to shared resources (e.g., rows of data) in concurrent environments. Two main strategies for handling concurrent updates to shared data are **optimistic locking** and **pessimistic locking**.

#### 1. **Optimistic Locking**

**Optimistic locking** assumes that **conflicts between transactions are rare**, and that multiple transactions can proceed without locking resources. Instead of locking data when it's read, optimistic locking only checks for conflicts when attempting to write (update) the data.

##### How It Works:
- **Read**: When a transaction reads a record, it doesn't lock it.
- **Update**: Before writing the updated data back to the database, the system checks whether another transaction has modified the data in the meantime (typically using a version number or timestamp).
- **Conflict Detection**: If another transaction has modified the record, a conflict is detected, and the current transaction is **rolled back** or retried.

##### Example:
- You have a record with a **version number** or **timestamp**.
- Transaction A reads the record with version 1.
- Transaction B reads the same record with version 1.
- Transaction A updates the record and increments the version to 2.
- Transaction B attempts to update the same record. Since the version has changed (it's now 2), the system rejects B's update, as the record was modified by A.

##### Pros of Optimistic Locking:
- **Better Performance**: No locks are held during reading, so it's more efficient in environments where conflicts are rare (e.g., read-heavy applications).
- **No Deadlocks**: Since no locks are held during reading, the chance of deadlocks is significantly reduced.
- **Scalability**: Works well in distributed systems with high contention for resources.

##### Cons of Optimistic Locking:
- **Retry Logic**: When conflicts are detected, the application must handle retries or rollbacks, which adds complexity.
- **High Conflict Rates**: In environments with frequent updates or high contention, optimistic locking can lead to a lot of retries, reducing overall efficiency.

##### When to Use:
- Optimistic locking is ideal for scenarios where:
  - There are **many reads** and **few writes**.
  - Contention for data is **low**.
  - Performance is prioritized over strict locking.

---

#### 2. **Pessimistic Locking**

**Pessimistic locking** assumes that **conflicts are likely**, so it locks the data as soon as a transaction reads it, preventing other transactions from modifying the data until the lock is released.

##### How It Works:
- **Read with Lock**: When a transaction reads a record, it locks the record, preventing other transactions from reading or modifying it.
- **Exclusive Lock**: The lock remains in place until the transaction is completed (commit or rollback). During this time, other transactions must wait or fail if they attempt to access the locked record.
- **Write**: After making changes, the transaction releases the lock.

##### Example:
- Transaction A reads a record and locks it.
- While A holds the lock, Transaction B cannot access the record.
- After A finishes and commits its transaction, the lock is released, and B can access the record.

##### Pros of Pessimistic Locking:
- **Conflict Avoidance**: Since data is locked as soon as it's read, there is no chance of conflicts or dirty reads (reading uncommitted data).
- **No Retry Logic**: Transactions don't need to be retried or rolled back due to versioning issues, as other transactions are simply blocked until the lock is released.

##### Cons of Pessimistic Locking:
- **Performance Bottleneck**: Transactions that involve locks can block other transactions, leading to poor performance in high-contention environments.
- **Deadlocks**: If two transactions lock different resources and wait for each other to release their locks, a deadlock can occur, requiring special handling by the database.
- **Scalability**: Locking resources for long periods makes it less scalable, especially in distributed systems.

##### When to Use:
- Pessimistic locking is ideal for scenarios where:
  - **Contention for resources is high**.
  - **Data integrity** is critical, and the cost of rolling back transactions is too high.
  - Applications are **write-heavy** or involve complex business logic with many updates.

---

### Key Differences

| Feature                 | Optimistic Locking                             | Pessimistic Locking                            |
|-------------------------|------------------------------------------------|------------------------------------------------|
| **Assumption**           | Conflicts are rare                             | Conflicts are likely                           |
| **Locking**              | No locks during read; checks at write time     | Locks data during read                         |
| **Performance**          | Higher performance with low contention         | Slower due to locks, especially in high contention |
| **Deadlocks**            | No risk of deadlocks                           | Can lead to deadlocks                          |
| **Conflict Handling**    | Detects conflicts at commit time, requires retries | Prevents conflicts with immediate locking      |
| **Use Cases**            | Read-heavy workloads, distributed systems      | Write-heavy workloads, high contention for data |

---

### Use Case Examples

- **Optimistic Locking**: 
  - A **web application** with high read traffic and occasional updates (e.g., e-commerce websites where many users are viewing products, but only a few users are updating the stock or making purchases).
  
- **Pessimistic Locking**:
  - **Banking systems** or **inventory systems** where simultaneous updates could cause data corruption or loss (e.g., updating account balances or product stock in real-time).

### Summary
- **Optimistic Locking** is better suited for environments with **low contention** and more **read operations**. It allows for better performance in distributed systems.
- **Pessimistic Locking** is more suitable for scenarios with **high contention** or **write-heavy operations**, where preventing conflicts is critical. However, it can lead to performance issues due to locking.
