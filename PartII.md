# Part II: Distributed Data

What happens if multiple machines are involved in storage and retrieval of data?

There are various reasons why you might want to distribute a database across multiple machines including:

*Scalability* 

If your data volume, read load, or write load grows bigger than a single machine can handle, you can potentially spread the load across multiple machines.

*Fault tolerance/high availability*

If your application needs to continue working even if one machine (or several machines, or the network, or an enture datacenter) goes down, you can use multiple machines to give you redundancy. When one fails, another one can take over.

*Latency* 

If you have users around the world, you might want to have servers at various locations worldwide so that each user can be served from a datacenter that is geographically close to them. That avoids the users having to wait for network packets to travel halfway around the world.

## Scaling to Higher Load

If all you need is to scale to higher load, the simplest approach is to buy a more powerful machine (sometimes called vertical scaling or scaling up). Many CPUs, many RAM chips, and many disks can be joined together under one operating system. This is also called *shared memory architecture*.

The problem is that the cost grows faster than linearly: a machine with twice the CPU, RAM, etc. typically cost more than twice the amount. And due to bottlenecks, a mechine twice the size cannot necessarily handle twice the load. This approach is also limited to one geographic location.

### Shared-Nothing Architectures

Shared-nothing architectures have gained a lot of popularity. Each machine or virtual machine running the database software is called a node. Each node uses its CPUs, RAM, and disk independently. Any coordination between nodes is done at the software level, using a conventoinal network.

No special hardware is needed, and you can potentially distribute systems in multiple geographic regions, and thus reduce latency for user and potentially be able to survive the loss of a datacenter.

### Replication Versus Partitioning

There are two common ways data is distrubuted across multiple nodes:

*Replication* 
Keeping a copy of the same data on several different bodes, potentially in different locations.  Replication provides redundancy: if some nodes are unavailable, the data can still be served from the remaining nodes.

*Partitioning*
Splitting a big database into smaller subsets called partitions so that different partitions can be assigned to different nodes. 

