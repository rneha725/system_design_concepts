## Storage Engines
---
Starting with a very simple idea of storage, say we have a file on disk and we have get and set functions on it:
```
get `key` -> gets the last value of the key by scanning the file
set `key:value` -> appends to the file
```

Writes: writes on this file will be very fast as we are appending to a file.
Reads: while reads might be slower as we need to scan the file to read a value.

To make reads faster, we can make use of indexes. Introducing indexes will make the reads faster but as the write operation now has 2 tasks: 1. append to the file; 2. Add/update the index, it makes writes slower.

### Indexes
#### Hash Indexes

