## Overview
It is an in-memory data structure storage system. String, Hash, List, Set, Sorted Set or ZSet and range query, Bitmaps, Hyperloglogs, and Geospatial index radius query. The common data structure types are String, List, Set, Hash, and ZSet.

## Features
- Supports various data structures
- built-in replication (Replication)
- LUA scripting (Lua scripting)
- LRU driver events (LRU eviction)-
- Transactions (Transactions) 
- Different levels of disk persistence (Persistence)
    - Redis uses something like a WAL to update the data. This WAL is flushed into the disk at regular intervals. As per my understanding this interval can be configued according to the needs. A less interval will provide more durability to the data and less write throughput and vice-versa. Also, flushing can be completely turned off to make it a good network data cache.
- Redis sentry and automatic partition (Cluster) to provide high availability

## Applications
- Cache
- Database
- message middleware
- Streaming system

## Redis and IO multiplexing and single-thread system
We use single thread system in Redis, generally to have a single-thread server, we have an event loop. Event loop has a simple lifecycle, a single-thread 
-> wait for connections on the socket -> accepts the connection -> executes the commands within -> wait -> accepts -> so on.
No overlap or concurrency issue here, but latency will take a hit.
Whereas a multi-thread server, listens on multiple sockets using threads and ask CPU to run a command while other threads can schedule their commands to be executed.
This requires using mutexes and semaphores.

When it comes to Redis, it takes the advantage of the fact that network IO is slower than the time it takes to execute the command in-memory. It uses, IO multiplexing to listen on multiple sockets and executes the commands whenever it sees data on any of the sockets. As it runs its commands in-memory, it is very fast.

Also, it can efficiently store the data structures, I could not understand it much but please refer to the blog post in the `Resource` section.


### Resource
- Arpit Bhayan on Redis: https://www.youtube.com/watch?v=h30k7YixrMo
- This very helpful blog: https://blog.devgenius.io/lets-dance-on-the-redis-floor-d93e02828bef