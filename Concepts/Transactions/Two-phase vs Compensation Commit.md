## Overview
Two phase commit(2PC) and try-commit/cancel(TCC, Saga architecture or compensation commit), both are used to address distributed transactions. 

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

### **Failure Scenarios in 2PC and Recovery Strategies**:

1. **Participant Failure**:
   - During **Prepare**: If a participant crashes before voting, it will either retry or abort when it recovers, depending on its transaction log.
   - During **Commit**: If a participant crashes after committing, it will use the log to ensure it completes the transaction on recovery.
   
2. **Coordinator Failure**:
   - **During Prepare**: If the coordinator crashes before gathering votes, participants remain blocked, holding locks.
   - **After Commit Decision**: If the coordinator fails before sending the commit/abort message, participants remain in the prepared state, risking resource blocking.

---

### **Applications of 2PC and When to Use**:
- **Banking Transactions**: Atomic money transfers between accounts.
- **Distributed Databases**: ACID-compliant transactions across distributed data nodes.
- **When to Use**: Strong consistency and atomic operations are critical; use 2PC when transaction integrity must be guaranteed.

---

### **What is TCC (Try-Confirm/Cancel)?**
- **TCC** is a pattern used in distributed systems where each action is split into three phases: **Try** (prepare resources), **Confirm** (commit if all actions succeed), and **Cancel** (rollback if any action fails). It ensures eventual consistency.

---

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

### **Diagram for 2PC and TCC**

