## Single Leader replication
- Single server takes the writes, followers take the reads. Leader can be for all writes or per partitions.
- adding a follower results in the copy the replication log from the leader.
- when follower crashes, it reboots from the point it crashed and reads the remaining replication log from the leader.
- leader crash: failover
  - problems: missing writes which we were not replicated, accidental failover (split brain)
- types: synchronous (strong consistency), async(eventual consistency)

## Eventually consistency
Apart from other problems
- reading your own writes, if read from replica and the write is not propagated
- monotinic reads: multiple clients are seeing differnt data
- consistent prefix read: if one write is dependant on the other, in the multi-leader or leaderless, write might fail or something else.

## Multi-leader replication
- clients can send their writes to multiple leaders
- multi-leader replication topology: circular, star, all-to-all, all have their tradeoffs, common one is **write conflicts**
- better comapred ti single leader for write throuputs
#### Conflict resolution
- Conflict avoidance: let a data always write to the same leader, might not work if this leader goes down.
- Last Write wins: latest lamport timestamp can be chosen, writes can be lost, clocks are not reliant (clock skew), if I make change to google docs offline, what timestamp that would be?
- on read: writes are done on the servers, on read, let the user manually merge them, version vectors used
  - version vector happens before: one version vector is strictly greater than all (most updated)
  - conflict: otherwise take max of both and repair, if not possible, then branching -> manually merged 
- on write: CRDT types

## Leaderless replication
- anti-entropy
- read-repair
- Quorum based: (w+r)>n
- two writes are writing to same key but different replica: write conflict.
- Sloppy quorum and hinted handoff

## Sharding/Partitioning
- key range division : simple but hotspot, easier for range queries.
- key hash division : use consistent hashing. Good hash => even distribution. Still hotspot.
- Maintain a global and optionally a secondary index.
- Dynamic paritioning might result into more network usage in case of faulty failure detenction.
- Only when it is absolutely necessary.
- Use coordinator or gossip protocol.
- Rebalance is complex.

## Isolation/Serializability
- It is too expernsice to provide full isolation so dbs have some optimizations
- Dirty reads: reading a non-committed write. sol: remember the old value.
- Dirty write: overwritting not committed data. sol: have a lock.
- Isolation levels:
  - read committed isolation: weakest. Prevents dirty reads and writes.
- Read skew: long reads, while reading rows change. sol: snapshot isloation. Snapshot isolation keeps a monotonically increasing transaction id. When rows are updated, a transaction id will get associated to it. When a read comes, we try to read the maximum value which is less than the read transaction id.
- Lost updates: two threads are trying to update a value by some number. They read and added 1000. If two threads read it at the same time, they might try to add the value and write to the row. => inconsistent state. Sol: atomic writes, explicit locking, automatic db detection
  - Atomic counters: shared variables between threads are updated only using atomic variables. For a single instance this can work better than locking, but for distrubuted systems, it can introduce latency and contention.
  - Locking: can lead to bugs
  - Automatic db detection: in conjuction with db snapshot, we can fail a write if it read some other value which is not updated by some other write.
  - These techniquees might not work for multi-leader or leaderless setup as these solutions assume one copy of data.
- Write skew: lock reading rows.
- <img width="300" alt="image" src="https://github.com/user-attachments/assets/7bb88942-2931-44b9-b309-fafc0e4be9cf">



## Concurrent writes
- Detecting concurrent write is all about what the client reads before the write.
