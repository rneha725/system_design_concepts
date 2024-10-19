[Video Resource](https://www.youtube.com/watch?v=gZP2VUmH05A)

- CRDTs are used to converge write conflicts, previously there are two ways to deal with the write conflicts
1. Use LWW
2. Let the client manually converge: Dynamo uses it
3. Using CRDTs

Please refer to the video contect for CRDTs. Overall, it is a data type which has two maps: increments and decrements, each node can insert a key-value pair inside the respective map and 
add a value to increment with a key(created by the node), and this delta of map is then shared accross, in case there is any conflict arise in one key, the mxa value is chosen as CRDT is monotinically
increasing. This way it offers strong eventual consistency.
