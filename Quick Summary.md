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

