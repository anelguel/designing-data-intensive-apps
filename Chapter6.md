# Chapter 6: Partitioning

Replication means having multiple copies of the same data on different nodes. For very large datasets that is not sufficient: we need to break the data up into partitions, also known as sharding. 

Each partition is a small database of its own, although the database may support operations that touch multiple parititions at the same time.

The main reson for wanting to partition data is scalability. Different partitions can be placed on different nodes on a shared-nothing cluster. Thus, a large dataset can be distributed across many disks, and the query load can be distributed across many processess.