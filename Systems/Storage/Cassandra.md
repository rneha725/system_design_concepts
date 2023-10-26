## Overview
---
Cassandra is a distributed database with high availability and it can manage heavy workloads. The requestor can connect to any node in the cluster and the system is able to serve the data. All the nodes remain in sync with the help of [[Gossip Protocol|gossip protocol]].
It is a key partitioned row data store.
Cassandra is a KKV store, 2 Ks are the primary key, first key is also used as the partition key and the second K is the clustering key.

https://news.ycombinator.com/item?id=28296507
In Cassandra, we have multiple nodes in a cluster. Data is distributed among these nodes. A node also has a replica of the data from the other node. A node can receive the read/write request for any data.

## How it serves the data?
---
As stated above, Cassandra has a cluster and it is generally contains multiple data centers and these datacenters in turn contain the nodes. Each node contains a piece of data for the cluster, as well as the replica for some other node(s). Whenever a request comes, any node can receive it and then it works as a coordinator and connects the request to the node which contains the relevant data but still the coordinator works as a proxy.
### Commit Logs
When there are write requests, it is first written to a WAL file called commit logs, it is then written to memtable and when memtable is full it is written to SS Table.

## Applications
----
Cassandra is developed to handle heavy workloads across multiple nodes without a single point of failure.

## AP over CP
---
Cassandra has a compaction process, which I think runs periodically to clear the tombstones. Tombstones are the marking on the data which needs to be deleted. 

### Master-less architecture

### Tombstones
As tombstones mark the data as 'to-be-removed', when there are a lot of tombstones in the systems, read queries might slow down as there are more data to go through to read the relevant data.
I think due to this, it is good for write-heavy systems as comapred to read-heavy systems.
Removing tombstones requires compaction, in case we want a system to have better read latencies (given that the system is still a write-heavy system), we need to run compaction more frequently.

### Compaction
---
Cassandra is known to have better writing speed as compared to read. [??]

### Questions
- Does casandra distributes data among the nodes, so no node contain all the data?

#gossip-protocol #cluster #commit-log #ss-table #bloom-filter #mem-table 