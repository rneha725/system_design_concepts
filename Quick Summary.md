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

## Concurrent writes
- Detecting concurrent write is all about what the client reads before the write.
