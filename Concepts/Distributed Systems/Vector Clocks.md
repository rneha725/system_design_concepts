https://medium.com/big-data-processing/vector-clocks-182007060193

Like Lamport’s Clock, Vector Clock is also a logical clock, which is used to assign timestamps for events in a distributed system. Vector clock also gives a partial ordering of the events. One of the shortcomings of Lamport’s clock is that it cannot identify concurrent events that are causally related. Here, instead of using integer values for the timestamp, we use a vector of integer values to represent the timestamp. If we have N processes in the group, then each process will have a vector with N elements.

The rule for incrementing the timestamp for the vector clock is also similar to Lamport’s clock with a small addition to it. The events update the logical timestamp according to the below rules:

Before executing an event, the process i increments the i-th element of its vector clock by 1
V[i] = V[i] + 1
2. During a send event, the process i increments the i-th element of its vector clock by 1 and sends the time(Vmsg[1…N]) along with the message.

V[i] = V[i] + 1
3. During a receive event, the process i increments the i-th element of its vector clock by 1. For all other processes, it takes the maximum of the corresponding element in the incoming message and its own local vector and sets it as the corresponding element in the local clock itself.

V[i] = V[i] + 1
V[j] = max(Vmsg[j], V[j]) for j ≠ i
We can use the vector timestamps to verify if two events are either causally related or concurrent. Before that, we will see how two vector timestamps are compared.

For two given timestamps V1 and V2,

V1 = V2 , iff V1[i] = V2[i], for all i = 1 to N (i.e, V1 and V2 are equal if and only if all the corresponding values in their vector matches)
V1 ≤ V2 , iff V1[i] ≤ V2[i], for all i = 1 to N 
Two events with vector timestamps V1 and V2 are causally related, if

V1 < V2 , iff  V1 ≤ V2 & there exists a j such that 1 ≤ j ≤ N & V1[j] < V2[j]
Two events with vector timestamps V1 and V2 are concurrent, iff

NOT (V1 ≤ V2 ) AND NOT (V2 ≤ V1 )

Fig 1: Vector Clock
The figure depicts how the vector timestamp values are incremented for different types of events. Let's see a few examples of concurrent events and causally related events.

Examples for Causally related events:

a -> j as (1,0,0) < (2,2,0)

a -> b as (1,0,0) < (2,0,0)

m -> k as (0,0,2) < (6,3,2)

Examples for Concurrent events:

For the events b and l we have vector timestamps (2,0,0) & (0,0,1) respectively. Here neither b ≤ l nor l ≤ b. Hence, they are concurrent events. Similarly, d and j are concurrent events.

Vector clock is also a logical clock that obeys causality and ensures partial order of events. Compared to Lamport’s clock, it can identify concurrent events efficiently as Vector Clock is represented using more space.

References:

Image: https://www.cs.rutgers.edu/~pxk/rutgers/notes/clocks/index.html

https://www.coursera.org/learn/cloud-computing/lecture/dy8wf/2-5-vector-clocks
