<sub>Note: these are some of the design choices which I collected while preparing for interviews. They might not be complete or sometimes be faulty. I will try to quote the reason while making a choice for a particular problem. It is not about building the whole system too, just how to solve the problem for the part of a bigger system.</sub>

<details><summary><h3>Problem: Read consistency</h4></summary>

-----
#### Problem
Read consistency for systems, like payment systems.

------

#### Part of the system we are solving for
While building a read consistent system, which architecture is better leaderless or leader based.

-------

#### Opinion 
A leader based architechture could be a better choice as we always know that one node(leader) always has the latest data and if we can direct the reads directly to it, we will have the consistent data. Although it also makes it SPOF. 

On the other hand, using a quorum based lederless architecture, we need to do read repairs(refer to dynamo [vector clocks](https://github.com/rneha725/system_design_concepts/blob/main/Systems/Storage/Dynamo%20Based%20Storage.md?plain=1#L23)) and for reading we need to satify the quorum which will also make it less performant. 

------
</details>
