A **fencing token** is a mechanism used in distributed systems to prevent outdated or concurrent operations from interfering with each other, especially in the context of **distributed locks** or **leases**.

### Purpose of Fencing Tokens:
Fencing tokens are typically used to avoid the "split-brain" problem, where two or more clients might believe they hold the same lock or lease due to network partition or clock skew, leading to race conditions or inconsistent state.

### How it Works:
1. **Lock/Lease Acquisition**: When a client (e.g., service, process) acquires a lock or lease, a **fencing token** (typically a monotonically increasing number) is issued by the lock service (e.g., distributed lock manager or database).
2. **Operation Execution**: When the client performs an operation that requires the lock (e.g., writing to shared resources), it sends the fencing token along with the request.
3. **Validation of Fencing Token**: The system receiving the operation (e.g., the service managing the shared resource) checks the fencing token to ensure it's higher than any previously seen token. This guarantees that only the most recent holder of the lock is allowed to perform operations.
4. **Preventing Race Conditions**: If a client with an older fencing token (e.g., an outdated lock holder due to a network partition) tries to perform an operation, the system can reject the request because the fencing token is lower than the latest one.

### Example:
Consider a distributed system where multiple clients can acquire a lock to update a shared resource (e.g., a file or database).

- Client A acquires a lock with fencing token `100` and starts writing to the resource.
- Due to a network partition, Client B is also granted a lock and receives fencing token `101`. Client A still believes it holds the lock.
- Client A tries to write to the resource with fencing token `100`. The resource manager rejects this operation because the token `100` is lower than the latest token `101` held by Client B.

### Use Cases:
- **Distributed Locks**: To prevent stale lock holders from performing operations after their lease has expired.
- **Lease-based Systems**: Where clients acquire a temporary right to a resource, and only the latest lease holder is allowed to modify the resource.
- **Concurrency Control**: In systems where multiple processes may concurrently perform operations, fencing tokens ensure that operations occur in the correct order and by the correct entities.

### Key Features of Fencing Tokens:
- **Monotonic**: The fencing tokens are monotonically increasing, ensuring that newer tokens always represent the most recent lock or lease.
- **Global Uniqueness**: The system issuing the tokens must ensure that they are unique and sequential across the system, to correctly order operations.
- **Stateless**: The system receiving operations (e.g., the resource being modified) does not need to store lock information but only needs to remember the latest fencing token it has accepted.

### Why Fencing Tokens are Important:
- **Avoiding Inconsistency**: Without fencing tokens, outdated or concurrent operations might corrupt shared resources.
- **Ensuring Safety in Distributed Systems**: In distributed environments, network partitions and delays are common, and fencing tokens help ensure that only valid operations succeed.

### Fencing Token Example in Distributed Locking (Zookeeper, etc.):
In systems like **Zookeeper** or **Etcd**, fencing tokens can be used in scenarios where clients acquire locks to update shared resources. Each time a client successfully acquires a lock, a unique fencing token is generated. When the client attempts to update a resource, it sends this token to ensure the resource manager only accepts operations from the valid lock holder.
