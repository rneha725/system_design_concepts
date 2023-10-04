Cassandra is a KKV store, 2 Ks are the primary key, first key is also used as the partition key and the second K is the clustering key.

https://news.ycombinator.com/item?id=28296507

Cassandra has a compaction process, which I think runs periodically to clear the tombstones. Tombstones are the marking on the data which needs to be deleted. 

Cassandra is known to have better writing speed as compared to read. [??]