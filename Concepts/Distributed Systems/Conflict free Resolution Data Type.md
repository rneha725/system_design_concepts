[Video Resource](https://www.youtube.com/watch?v=gZP2VUmH05A)

- CRDTs are used to converge write conflicts, previously there are two ways to deal with the write conflicts
1. Use LWW
2. Let the client manually converge: Dynamo uses it
3. Using CRDTs

Please refer to the video contect for CRDTs. Overall, it is a data type which has two maps: increments and decrements, each node can insert a key-value pair inside the respective map and 
add a value to increment with a key(created by the node), and this delta of map is then shared accross, in case there is any conflict arise in one key, the mxa value is chosen as CRDT is monotinically
increasing. This way it offers strong eventual consistency.

---- 

Better resource : https://www.youtube.com/watch?v=B5NULPSiOGw&t=2151s

This is by MArtin Klepman, and he has explained the low level detail of the CRDT, only resource I could find where I understaood the concept finally.

### A Simple use case:

<img width="1052" alt="image" src="https://github.com/user-attachments/assets/11677452-9c26-4608-8cb9-0734d5946b8e">

### An insertion 

<img width="1054" alt="image" src="https://github.com/user-attachments/assets/e871c643-5479-4d71-9e75-002169488b73">


