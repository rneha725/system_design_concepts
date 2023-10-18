Casandra is a key partitioned row data store.
Cassandra is a KKV store, 2 Ks are the primary key, first key is also used as the partition key and the second K is the clustering key.

https://news.ycombinator.com/item?id=28296507

### AP over CP

Cassandra has a compaction process, which I think runs periodically to clear the tombstones. Tombstones are the marking on the data which needs to be deleted. 

### Master-less architecture

### Tombstones
As tombstones mark the data as 'to-be-removed', when there are a lot of tombstones in the systems, read queries might slow down as there are more data to go through to read the relevant data.
I think due to this, it is good for write-heavy systems as comapred to read-heavy systems.
Removing tombstones requires compaction, in case we want a system to have better read latencies (given that the system is still a write-heavy system), we need to run compaction more frequently.


### Compaction

Cassandra is known to have better writing speed as compared to read. [??]

### Questions
- Does casandra distributes data among the nodes, so no node contain all the data?