## Overview

Dynamo is a distrubuted kv storage architecture, it is designed to be **highly available**, scalable, eventually consistent, persistent(durable) and completely decentralized.

Availbility is the preferred over consistency and flexibility is provided for consistency. Dynamo is also called zero-hop DHT (Disctributed Hash Table), as it forwards the client request to the right node(coordinator node) using the cluster-aware service.

The architecture has influenced cassandra, dynamdb, voldemort etc.

## Uses

- **why is it developed**: to create a highly avaialable storage and although it can also provide high consistency(with performance compromises), it is not suitable for highly consistent use cases.
- **Use Cases**: 

## How it works?

Dynamo is made highly available by introducing redundancy and handling node failires.

### Key points

- Uses [consistent hashing](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Consistent%20Hashing.md) to serve the request and key distribution
- Stores replicas in backup nodes
- Uses [sloppy quorum and hinted handoff](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Consistent%20Hashing.md) to serve requests, this makes dynamo based systems to be available even during the node failures -> **"always writable"**
- Uses [vector clocks](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Vector%20Clocks.md) to converge versions of the values for a key
    - If cannot be resolved due to branching, it is sent back to client to resolve, simiular to git merge.
    - Conflict is resolved during the reads (Read Repair), once client chooses a value, it is sent back to the dynamo service and back to the replicas not having this updated value.
    - Example: shopping cart values, merged will be provided, so even the client has deleted an item, in the resolved cart, the item will be present.
    - Vectors are truncated if they pass over a limit, this can result in value not getting resolved, but this problem is not faced uptil now as per the creators of this universe.
- Uses [LWW](https://dzone.com/articles/conflict-resolution-using-last-write-wins-vs-crdts) -> like a coin toss
- Clients can use a partition-aware client library that routes the requests to the appropriate coordinator nodes with lower latency. -> zero-hop DHT
    ![image](https://github.com/user-attachments/assets/86820a49-69aa-4e9f-abc0-e59d46c84f2c)
- Uses state machine to handle request
- write operation as stated below contains a context, this also stores the node which replied fastest to the given key, when the get was performed(usually a write is performed after get). During the write operation, using this context the coordinator node is chosen.
- In addition to vector clocks, [merkle trees](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Merkle%20Trees.md) are used to converge values in the replicas. Merkle trees are hash based binary tree and use divide and conquer on a hash value of the data present in a replica, hence we know the location of different data in primary and the replica.
- Uses [gossip protocol](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Gossip%20Protocol.md) to have the infoirmation of about health nodes.

### APIs

#### **get(key)** 
The get operation finds the nodes where the object
associated with the given key is located and returns either a single
object or a list of objects with conflicting versions along with a context .
The context contains encoded metadata about the object that is
meaningless to the caller and includes information such as the version
of the object (more on this below).

Dynamo’s get() request will go through the following steps:
1. The coordinator requests the data version from N − 1 highest-ranked
healthy nodes from the preference list.
2. Waits until R − 1 replies.
3. Coordinator handles causal data versions through a vector clock.
4. Returns all relevant data versions to the caller.

#### **put(key, context, object)**
The put operation finds the nodes where
the object associated with the given key should be stored and writes the
given object to the disk. The context is a value that is returned with a
get operation and then sent back with the put operation. The context
is always stored along with the object and is used like a cookie to verify
the validity of the object supplied in the put request.

Dynamo’s put() request will go through the following steps:
1. The coordinator generates a new data version and vector clock
component.
2. Saves new data locally.
3. Sends the write request to N − 1 highest-ranked healthy nodes from
the preference list.
4. The put() operation is considered successful after receiving W − 1
confirmation.

A preference list is the N healthy nodes in the consistent hash ring. Hinted handoff is used if the main server is unavailable.

#### Cassandra and Riak:

![image](https://github.com/user-attachments/assets/0c81f5e6-3aa5-411e-93a7-5db6e87cb55b)

## Summary:
![image](https://github.com/user-attachments/assets/d5949abf-5309-4680-8ffd-7edc57120b39)

## Pros and Cons

## Resources
- Notes resource: [Grokking the Advanced System Design Inyterview](https://github.com/rneha725/Books/blob/main/Compressed%20Grokking%20the%20Advanced%20System%20Design%20Interview-compressed.pdf)
- [Sloppy Quorum and Hinted Handoff - distributed computed musings](https://distributed-computing-musings.com/2022/05/sloppy-quorum-and-hinted-handoff-quorum-in-the-times-of-failure/)
