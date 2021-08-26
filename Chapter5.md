# Chapter 5: Replication

*Replication* means keeping a copy of the same data on multiple machines that are connected via a network.  There are several reasons why'd we do this including:

* To keep data geographically close to your users (and thus reduce latency)
* To allow the system to continue working even if some of its parts have failed (and thus increase availability)
* To scale out the number of machines that can serve read queries (and thus increase read throughput)

Replicaion lies in handling changes to repicated data (compared to a static dataset). There are three popular algorithms for replicating changes between nodes: single-leader, multi-leader, and leaderless.

## Leaders and Followers

Each node that stores a copy of the database is called a replica. With multiple replicas, the question inevitably arises: how do we ensure that all the data ends up on all the replicas?

Every write to the database needs to be processed every replica, otherwise the replicas would no longer contain the same data. The most common form of this is called leader-based replication (or active/passive).

1. One of the replicas is designated the leader. When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.

2. Whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of the replication log or change stream. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all writes in the same order as they were processed on the leader.

3. When the client wants to read from the database, it can query either the leader or any of the followers. However, writes are only accepted on the leader (the followers are read-only from the client's point of view).

## Synchronous Versus Asychronous Replication

*Asynchronous*: Not waiting for something to complete (e.g., sending data over the network to another node), and not making any assumptions about how long it is going to take. In this asynchronous means the leader sends the message, but doesn't wait for a response from the follower.

*Synchronous* is the opposite, and in this case, it means that the leader waits until the follower has confirmed that it recieved the write before reporting success to the user, and before making the write visible to other clients.

## Setting Up New Followers

Setting up new followers may be done perhaps in order to increase the number of replicas or to replace failed nodes. How do you ensure that the new follower has an accurate copy of the leader's data?

Conceptually the process looks like this:

1. Take a consistent snapshot of the leader's database at some point in time - if possible without taking a lock on the entire database. Most databases have this feature, as it is also required for backups.

2. Copy the snapshot to the new follower node.

3. The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader's replication log.

4. When the follower has processed the backlog of data changes since the snapshot, we say it has caught up. It can now continue to process data changes from the leader as they happen.

## Handling Node Outages

Any node in the system can go down, perhaps unexpectantly due to a fault, but just as likely due to planned maintenance. Being able to reboot individual nodes without downtime is a big advantgae for operations and maintenance. Thus, our goal is to keep the system as a whole running despite individual node failures, and to keep the impact of a node outage as small as possible.

How to achieve high availability with leader-based replication?

### Follower failure: Catch-up recovery

On its local disk, each follower keeps a log of the data changes it has received from the leader. If a follower crashes and is restarted, or if the network between the leader and the follower is temporarily interuppted, the follower can recover quite easily: from its log, it knows the last transaction that was processed before the fault occured. Thus, the follower can connect to the leader and request all the data changes that occurred during the time when the dollower was disconnected.

### Leader failer: failover

When a leader fails, recovering is trickier: one of the followers need to be promoted to be the new leader, clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. This process is called failover.

Failover can happen manually or automatically. And automatic failover usually consists of the following steps:

1. Determining that the leader has failed.

2. Choosing a new leader. This can be done different ways, typicallu the nest candidate for leadership is usually the replica with the most up-to-date data changes from the old leader.

3. Reconfiguring the system to use the new leader.

## Multi Leader Replication

A natural extension of the leader-based replication model is to allow more than one node to accept writes. Replication still happens in the same way: each node that processes a write must forward that data change to all the other nodes. We call this a multi-leader configuration. 

### Use Cases for Multi-Leader Replication

It rarely makes sense to use a multi-leader setup within a single datacenter, because the benefits rearely outweigh the added complexity. However, there are some situations in which this configuration is reasonable.

### Multi-datacenter operation

Imagine you have a datacenter with replicas in several different datacenters (perhaps so thatyou can tolerate failure of an entire datacenter, or perhaps in order to be closer to you users). With a normal leader-based replication setup, the leader has to be in one of the datacenters, and all writes must go through that datacenter.

In multi-leader configuration, you can have a leader in *each* datacenter. Within each datacenter, regular leader-follower replication is used; between datacenters, each datacenter's leader replicates its changes to the leaders in other datacenters.