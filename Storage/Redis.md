- [ ] Notes: https://blog.devgenius.io/lets-dance-on-the-redis-floor-d93e02828bef


## Overview
---
It is an in-memory data structure storage system. String, Hash, List, Set, Sorted Set or ZSet and range query, Bitmaps, Hyperloglogs, and Geospatial index radius query. The common data structure types are String, List, Set, Hash, and ZSet.

## Working
- It is atomic in nature
    - Provides thread safety
- Data is stored in-memory
    - a big reason it is used as a cache
- Logs every update operation: append only logs

## Features
---
- Supports various data structures
- built-in replication (Replication)
- LUA scripting (Lua scripting)
- LRU driver events (LRU eviction)
    - TTL on keys
    - eviction policy is configurable
- Transactions (Transactions)
    - 
- Configurable Persistence
    - Redis uses something like a WAL to update the data. This WAL is flushed into the disk at regular intervals. As per my understanding this interval can be configued according to the needs. A less interval will provide more durability to the data and less write throughput and vice-versa. Also, flushing can be completely turned off to make it a good network data cache.
- Redis sentry and automatic partition (Cluster) to provide high availability

## Applications
----
- Cache
- Database
- message middleware (pub/sub)
- Streaming engine