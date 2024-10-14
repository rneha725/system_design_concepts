## Overview
Two phase commit(2PC), try-commit/cancel(TCC), Saga are ways of dealing with distributed transactions.

For example, say we want to transfer balance from one wallet to another. In this we need to add an amount to one account and debit from other, and this needs to be done in one transaction. When data is not one single machine, transaction will be done on two different db servers. 

### **Notes on Prepare and Commit in Databases**

#### **Prepare and Commit in Databases**:
- **Prepare**: During the prepare phase in distributed databases, participants check if they can commit a transaction by locking necessary resources and writing to transaction logs. This ensures readiness to commit.  
- **Commit**: After all participants confirm readiness (in the "prepare" phase), the coordinator sends a commit message, and participants finalize the transaction, releasing locks and applying changes to the database. If any participant is unable to commit, the transaction is aborted.

---

### **What is Two-Phase Commit (2PC)?**
- **2PC** is a distributed algorithm that ensures atomicity (all-or-nothing) across multiple nodes or services in a distributed system. It operates in two distinct phases: **Prepare** and **Commit/Abort**.
  
---

### **Low-Level Description of Each Step in 2PC**:

1. **Prepare Phase**:
   - **Coordinator** sends a prepare message to all participants.
   - **Participants** lock resources, write logs indicating readiness, and respond with a "Yes" (ready to commit) or "No" (cannot commit).
   
2. **Commit/Abort Phase**:
   - If all participants vote "Yes", the coordinator sends a commit message. Each participant commits the transaction.
   - If any participant votes "No", the coordinator sends an abort message, and participants roll back the changes.

---

![image](https://github.com/user-attachments/assets/77ae3ef8-0b8a-484e-826d-6f60babe94b6)


### **Failure Scenarios in 2PC and Recovery Strategies**:

1. **Participant Failure**:
   - During **Prepare**: If a participant crashes before voting, it will either retry or abort when it recovers, depending on its transaction log.
   - During **Commit**: If a participant crashes after committing, it will use the log to ensure it completes the transaction on recovery.
   
2. **Coordinator Failure**:
   - **During Prepare**: If the coordinator crashes before gathering votes, participants remain blocked, holding locks.
   - **After Commit Decision**: If the coordinator fails before sending the commit/abort message, participants remain in the prepared state, risking resource blocking.

The resource locking can be prevented by having a timed lock on the resources. This way, if coordinator fails at any step, it is made sure that the lock will be released if the commit does not happen timely.

---

### **Applications of 2PC and When to Use**:
- **Banking Transactions**: Atomic money transfers between accounts.
- **Distributed Databases**: ACID-compliant transactions across distributed data nodes.
- **When to Use**: Strong consistency and atomic operations are critical; use 2PC when transaction integrity must be guaranteed.

### Trade offs
Pros:
- simple to implement
- strong consistency
- C > A
- guarantees atomocity

Cons:
- Latency and perfomance
- coordinator is a SPOF
- C > A
- Increased resource usages, as we are locking everything in one step
- limited scalability

### Alternative to 2PC:
- 3PC:
- Raft or Paxos
- eventual consistency
  
---

### **What is TCC (Try-Confirm/Cancel)?**
- **TCC** is a pattern used in distributed systems where each action is split into three phases: **Try** (prepare resources), **Confirm** (commit if all actions succeed), and **Cancel** (rollback if any action fails). It ensures eventual consistency.

---

![Screenshot 2024-10-13 at 6 31 07 PM](https://github.com/user-attachments/assets/685e2ca2-a2eb-4d71-bd6c-c91d2f276b02)


### **Low-Level Description of Each Step in TCC**:

1. **Try Phase**:
   - Each service performs a tentative action (e.g., reserving resources) and locks them.
   
2. **Confirm Phase**:
   - If all tentative actions succeed, services confirm the transaction and finalize changes.
   
3. **Cancel Phase**:
   - If any part of the transaction fails, compensating actions (e.g., releasing reservations) are performed to cancel the transaction.

---

### **Failure Scenarios in TCC and Recovery Strategies**:

1. **Failure During Try**:
   - Retry the Try phase or invoke the Cancel phase for rollback.
   
2. **Failure During Confirm**:
   - Confirm operations are idempotent, so retries can ensure the transaction is finalized.
   
3. **Failure During Cancel**:
   - Cancel operations should also be idempotent, and retrying ensures compensating actions (e.g., refunds) are performed correctly.

---

### **Applications of TCC and When to Use**:
- **E-commerce**: Reserving stock, confirming payment, and managing inventory.
- **Microservices**: Coordinating actions across multiple independent services.
- **When to Use**: Use TCC when eventual consistency is acceptable and you need non-blocking, fault-tolerant transaction handling.

---

## Saga

### Saga Pattern in Distributed Transactions

The **Saga** pattern is an alternative approach to handling long-running distributed transactions that require multiple microservices or databases to be updated. Unlike 2PC, which ensures atomicity by locking resources and waiting for a global commit/abort, Sagas achieve eventual consistency by breaking down the distributed transaction into a series of smaller, local transactions, each with compensating actions in case of failures. This pattern provides better availability and resilience to failures, especially in systems that prioritize availability over strict consistency.

---

### How Saga Works

A **Saga** is a sequence of local transactions where each transaction is independent and committed to its own database. If one transaction fails, a compensating transaction is executed to undo the effects of the previous transactions. The overall process can be considered either **choreographed** (where services communicate directly with one another) or **orchestrated** (where a central coordinator directs the flow of the Saga).

#### Phases in a Saga

1. **Execution of Local Transactions**:
    - Each service executes a local transaction that updates its own data store. Once it commits the transaction, the Saga moves to the next step.
    - The services do not require coordination or knowledge of the global transaction state, reducing the need for locks or coordination across multiple services.

2. **Compensation on Failure**:
    - If a local transaction fails, the Saga executes compensating transactions to undo the changes made by previous steps. Each service involved must define how to roll back its changes in case of a failure (e.g., issuing a refund for a canceled order).
    - Compensation is done in reverse order, starting from the last successful step and working backward.

---

### Example

Let’s assume a distributed transaction where a flight booking service, hotel booking service, and payment service are involved. A Saga would handle the process as follows:

1. **Book Flight**: The Saga starts by booking a flight. If successful, the system moves to the next step.
2. **Book Hotel**: Once the flight is booked, the Saga continues by booking a hotel. If successful, it proceeds.
3. **Process Payment**: The final step is to process the payment. If payment fails, compensating actions are triggered.
4. **Compensation**: In case of a failure at any stage, compensating actions are initiated, like canceling the hotel booking and flight booking if the payment fails.

---

### Trade-offs in Saga vs Two-Phase Commit (2PC)

| **Aspect**                | **Saga**                                          | **2PC**                                         |
|---------------------------|--------------------------------------------------|-------------------------------------------------|
| **Availability**           | High availability. Sagas avoid long-term locks, allowing services to remain available even during failures. | Lower availability due to potential blocking of resources during the prepare phase. |
| **Consistency**            | Achieves eventual consistency; services may see temporary inconsistencies during a saga. | Provides strong consistency; either all operations succeed or all are aborted. |
| **Failure Handling**       | Handles failures through compensating transactions. Each step must define a rollback procedure. | Abort phase rolls back all changes, but failures during the commit phase can cause blocking. |
| **Complexity**             | Requires defining compensating actions for each step. Transaction boundaries are more flexible. | Requires coordination across participants. Failure handling can lead to complex blocking and coordination problems. |
| **Resource Locking**       | No locks held across services during the entire transaction, reducing contention. | Locks resources during the prepare phase, potentially causing bottlenecks. |
| **Use Case**               | Ideal for long-running transactions where services can handle eventual consistency and temporary failures. | Suitable for distributed systems requiring strict atomicity and consistency. |

---

### Key Characteristics of Saga Pattern

1. **Decentralized Coordination**:
    - Sagas rely on either **orchestration** or **choreography** to coordinate between services. 
    - In **orchestration**, a central coordinator controls the flow of transactions, while in **choreography**, each service triggers the next service in the chain without a centralized controller.

2. **Eventual Consistency**:
    - Unlike 2PC, where strict consistency is maintained, Sagas allow for eventual consistency, meaning that services may temporarily hold inconsistent data until the entire Saga completes.
    - This is acceptable in many real-world scenarios where consistency can be relaxed (e.g., e-commerce orders, where confirming inventory may take time).

3. **Compensating Actions**:
    - A key feature of Sagas is the ability to undo steps via compensating actions. This rollback is necessary when one of the steps in the Saga fails.
    - Compensating transactions are business logic-driven (e.g., issuing a refund, restocking an item) rather than purely database-driven.

4. **Fault Tolerance**:
    - Sagas are more fault-tolerant than 2PC in many ways. If a service crashes, it can restart the local transaction or invoke a compensating transaction without affecting the entire system.
    - There is no central coordinator that could act as a single point of failure (in the case of choreographed Sagas).

---

### Advantages of Sagas

1. **No Global Locking**: Each service is responsible for its own local transaction, so there are no global locks across services. This improves the system's performance and availability, as services are not blocked waiting for other services to commit.
   
2. **Failure Isolation**: When a failure occurs, only the failed service is affected. The system can attempt to retry the failed transaction or invoke compensating transactions without affecting the other services.

3. **Scalability**: Sagas are highly scalable due to their decentralized nature. Each service can execute independently without waiting for a global commit decision.

4. **Resilience**: Sagas handle transient failures well. If a service is down, the Saga can retry or wait for the service to recover.

---

### Disadvantages of Sagas

1. **Compensation Complexity**: One of the most significant challenges with Sagas is defining compensating actions for each step. These compensating transactions can be complex, especially when undoing side effects like sending emails, updating external systems, or processing payments.
   
2. **Eventual Consistency**: Sagas guarantee eventual consistency, but this can be a drawback for systems that require strict, immediate consistency. There is a window of time where the system may be in an inconsistent state.

3. **No Atomicity Guarantee**: Since Sagas do not guarantee atomicity (all-or-nothing transactions), you must design each step to handle partial completion. If compensating actions fail or are not sufficient, the system may end up in a partially completed state.

4. **Out-of-Order Execution**: In some cases, steps of a Saga may not execute in the exact order expected, especially in choreographed Sagas. This can lead to unexpected results if not carefully managed.

---

### Conclusion

The Saga pattern is a powerful alternative to 2PC in distributed systems that prioritize availability and fault tolerance. By breaking down distributed transactions into smaller, local steps and using compensating actions to handle failures, Sagas provide a way to achieve eventual consistency without locking resources. However, the complexity of compensating transactions and the lack of atomicity make Sagas more suitable for business operations that can tolerate temporary inconsistencies rather than systems requiring strict transactional guarantees.



