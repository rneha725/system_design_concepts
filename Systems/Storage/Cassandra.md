## Overview
---
Cassandra is a distributed wide column NoSql database with high availability, scalable, reliable and it can manage heavy workloads. Cassandra combines disctributed nature of [Amazon's Dynamo](https://github.com/rneha725/system_design_concepts/blob/main/Systems/Storage/Dynamo%20Based%20Storage.md) which is a highly available kv store; and the data model of Gooogle's BigTable.

https://news.ycombinator.com/item?id=28296507

In Cassandra, we have multiple nodes in a cluster. Data is distributed among these nodes. A node also has a replica of the data from the other node. A node can receive the read/write request for any data.

## How it serves the data?
---

The requestor can connect to any node in the cluster and the system is able to serve the data. All the nodes remain in sync with the help of [Gossip Protocol](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Gossip%20Protocol.md).
It is a key partitioned row data store.

As stated above, Cassandra has a cluster and it is generally contains multiple data centers and these datacenters in turn contain the nodes. Each node contains a piece of data for the cluster, as well as the replica for some other node(s). Whenever a request comes, any node can receive it and then it works as a coordinator and connects the request to the node which contains the relevant data but still the coordinator works as a proxy.

## Applications
----
Cassandra is developed for high availability and to handle heavy workloads across multiple nodes without a single point of failure. It is optimized for higher throuputs and faster writes.

## Use cases:
- As key value storage of high availability
- Time series data modelling
- For write heavy applications

## How it works
- Data partitioning: Uses [consistent hashing](https://github.com/rneha725/system_design_concepts/blob/main/Systems/Storage/Dynamo%20Based%20Storage.md#:~:text=Uses-,consistent%20hashing,-to%20serve%20the) for data paritioning
- Replication: Replication is done within the cluster and across the cluster. Intra-cluster we have replication factor, which copies the data to next n - 1 nodes within the cluster. For inter-cluster, a request is forwarded to the other data center, this cluster uses its own relication factor.
- p2p system, master less: Casssandra is a p2p distributed system, all nodes are equal and there is no single point of failure.
- Consitetency Model: tunable consistency. When the cluster cannot meet the consistency levelspecified by the client, Cassandra fails the write request and does not store a hint. Snitch Component: It is responsible for getting the fastest replica which can respond. It has the information of all the network topology.
- Uses [gossip protocol](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Distributed%20Systems/Gossip%20Protocol.md) to detect any failures, we have seed nodes which are new nodes and more discoverble in the cluster so that the node quickly becomes part of this arrangement. Here, Cassandra also uses heatbeats, but the algorithm is adjusted according to the past records of the nodes and the timeout of heatbeat is decided on the past and other network failures. Uses generation number along with the node info, a generation number for a node monotonically increases when it restarts.
- Storage:
  - write operation: Cassandra's storage is done on a commit log (durability and creash recovery), memTable and SSTable with the compaction running periodically.
  - read operation:
    - row cache: contains the cache for whole row
    - Bloom Filters: on SSTable
    - key cache: cache to the offset of the key in SSTable
    - Partition Summary file: an another range index on the partition file (only on keys), stored in memory
    - Partition index file: stored on disk, index info on the SSTable
    <img width="653" alt="image" src="https://github.com/user-attachments/assets/f2010672-31fd-46a3-ac80-4b1fb5755de6">
- Compaction: Compaction is done on SSTable to make the size smaller and hence reads faster.
- Tombstones: When a delete happens, Cassandra updated the row as deleted by using extra memeory. These tombstones have default expiry of 10 days, and they are removed during compaction. They cause extra memory and might slow down read queries. They use extra memory.

### Data Partitioning:
Cassandra's primary key is used to identofy a row, primary key has two parts: 

<img width="400" alt="image" src="https://github.com/user-attachments/assets/ce903be2-6c2f-4e09-b106-f5de55f0d110">

- Partition key: It is used to identify the node where the key is stored. A hash fucntion uses this part to identify the node for the key.
- Clustering key: Data is sorted within a node using this key. There can be multiple clustering key and sorting happens in a cascade nature for these keys.

## Consistency Model
**write:** It has different types of consistency model for writes, refer to the educative book for it. With other normal consistency model where we are writing to a fixed number of replicas within and out of data centers, we have 'any' as consistency level, where a write can be performed on any node, in case the target node is down. This write is saved in hints file on the node and it is not used for reads. When the failed node comes back, this node pushes the value to the target node. Cassandra, keeps these hints for 3 hours only and after that deletes the data, this is useful in cases when a node is in the failed state for sometime and it will get a lot of write requests when it comes back again. Although, this also means that now the resurrected node will have the stale data. Cassandra relies on **read-replairs** for this.

**reads**: Reads are also responsible for read-repairs. Snitch is a component reponsible for forwearding the read requests to fastest nodes, the assigned node then do the read-repairs on all the replica nodes. To see if the data mismatches on different machine, [checksum](https://github.com/rneha725/system_design_concepts/blob/main/Concepts/Checksum.md) is used.

<img width="1510" alt="image" src="https://github.com/user-attachments/assets/a65d63b2-c054-4cd5-bede-08632192f8fa">

## Storage
Uses Google's BigTable like storage.
- Commit log: it is crash recovery mechanism in cassandra, every write in a node is first done on a WAL file, only after this it goes to memtable, if the write fails here, the write as a while is failed for that node.
- MemTable: This is an in-memory storage, uses partition key and clustering key for storage. Once full, it is flushed to SSTable(Sorted String Table), to flush the values to the disk.
- SSTable: Sorted String Table, this is an on-disk immutable storage. A read is done on both, memtable and SSTable. One table's data goes into one SSTable.

<img width="1143" alt="image" src="https://github.com/user-attachments/assets/9d375a4c-dc55-48aa-ab58-2213fedec851">

#### Definition of coordinator node
Unlike dynamo architecture, client does not have info about the cluster, a client can connect to any node in the cluster, this is called coordinator node. This node then helps identify the nodes reponsible for the key and forwards the request to those.

### Tombstones
As tombstones mark the data as 'to-be-removed', when there are a lot of tombstones in the systems, read queries might slow down as there are more data to go through to read the relevant data.
I think due to this, it is good for write-heavy systems as comapred to read-heavy systems.
Removing tombstones requires compaction, in case we want a system to have better read latencies (given that the system is still a write-heavy system), we need to run compaction more frequently.

### NoSql
Cassandra is a NoSQL database which means we cannot have joins between tables, there are no foreign keys , and while querying, we cannot add any column in the where clause other than the primary key. These constraints should be kept in mind before deciding to use Cassandra.

## Pros and Cons
