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

Imagine you have a datacenter with replicas in several different datacenters (perhaps so that you can tolerate failure of an entire datacenter, or perhaps in order to be closer to you users). With a normal leader-based replication setup, the leader has to be in one of the datacenters, and all writes must go through that datacenter.

In multi-leader configuration, you can have a leader in *each* datacenter. Within each datacenter, regular leader-follower replication is used; between datacenters, each datacenter's leader replicates its changes to the leaders in other datacenters.

### Clients with offline operation

Multi-leader replication is appropiate if you have an application that needs to continue to work while it is disconnected from the internet.

> Example: Consider the calendar apps on your mobile phone, and other devices. You need to see you meetings (make read requests) and enter new meetings (make write requests) at any time, whether you're connected to the internet or not. If you make changes and you're offline, they need to be synced with a server and your other devices when the device is next online. In each case, every device has a local database that acts as a leader (accepting write requests), and there is an asynchronous multi-leader replication process (sync) between the replicas of your calendar on all your devices. *From an archtectual point of view, this setup is essentially the same as multi-leader replication between datacenters. Each device is a "datacenter"*

### Collaborative Editing
*Real-time collaborative editing* applications allow several people to edit a document simultaneously like Google Docs. When one user edits a document, the changes are instantly applied to their local replica and asynchronously replicated to the server and any other users that are editing the same document.

If you want to guarantee no editing conflicts, the application must obtain a lock on the document before a user can edit it. If another user want to edit, they must wait for commited changes of the first user. This would be single-leader replication.

For faster replication, you'd want multi-leader replication where various people are working on the document at the same time, but that does pose greater conflicts.

## Handling Write Conflicts 
The biggest problem with multi-leader replication is that write conflicts can occue, which means that conflict resolution is required.

In the image below, user 1 changes the title of a page from A to B, and user 2 changes the title from A to C at the same time. Each user's change is successfully applied to the local leader. However, when changes are asynchronously replicated, a conflict is detected. This doesn't occur in single-leader databases.

![](images\chapter5\pg171.jpg)
![](https://github.com/anelguel/designing-data-intensive-apps/blob/main/images/chapter5/pg171.jpg) 

### Synchronous vs. Asynchronous conflict detection
In a single-leader databse, the second writer will either block and wait for the first write to complete. 

On the other hand, in a multi-leader setup, both writes are successful, and the conflict is only detected asynchronously at a later time (in which it may be too late to ask the user to resolve the conflict).

In principle, you could make the conflict detection *synchronous* - i.e. wait for the write to be replicated to all the replicas before telling the user that the write was successful. But then you lose the main advantgae of multi-leader replication: allowing each replica to accept writes independently. If you want synchronous conflict detection, you might as well use single-leader replication. 

### Converging toward a consistent state
A single leader database applies writes in sequencial order: if there are several updates to the same field, the last write determines the final value of the field.

In a multi-leader configuration, there is no defined ordering of writes, so it's not clear what the final value should be. In the figure above, at Leader 1 the title is updated to B and then C; at Leader 2 it is first updated to C then B. Neither order is "more correct" than the other.

If each replica simply applied writes in the order that it saw the writes, the database would end up in an inconsistent state: the final value would be C at leader 1 and B at leader 2. That is not acceptable - every replication scheme must ensure that the data is eventually the same in all replicas. 

Thus, the database must resolve the conflict in a *convergent* way, which means that all replicas must arrive at the same final value when all changes have been replicated.

There are various ways of achieving convergent conflict resolution:

* Give each write a unique ID (e.g. a timestamp, hash of a key and value, etc.), pick the write with the highest ID as the *winner*, and throw away the other writes. If a timestamp is used, this method is called *last write wins*. Although this approach is popular, it is prone to data loss.

* Give each replica a unique ID, and let writes that originated at a higher numbered replica always take precedence over writes that originated at a lower-numbered replica. This approach also implies data loss.

* Somehow merge the values together - e.g. order them alphabetically and then concatenate them (in Figure 5-7, the merged title might be something like "B/C")

* Record the conflict in an explict data structure that preserves all information, and write application code that resolves the conflict at some later time (perhaps by prompting the user).

### Custom conflic resolution logic
Most multi-leader replication tools let you write conflict resolution logic using application code, that code may be executed on write or on read.

### Multi-Leader Replication Topologies
A replication topology describes the communication paths along which writes are propagated from one node to another. With more than two leaders, various different topologies are possible.

![](images\chapter5\pg175.jpg)
![](https://github.com/anelguel/designing-data-intensive-apps/blob/main/images/chapter5/pg175.jpg) 

The most general is the all-to-all, where every leader sends its writes to every other leader. Circular topologies are where each node receives writes from one node and forwards those writes (plus any writes of its own) to one other node. A star topology is where one designated root node forwards writes to all of the other nodes. The star topology can be generalized to a tree.

In circular and star topologies, a write may need to pass through several nodes before it reaches all replicas. Therefore, nodes need to forward data changes they recieve from other nodes. A problem with circular and star topologies is that if just one node fails, it can interrupt the flow of replication messages between other nodes, causing them to be unable to communication until the node is fixed.

On the other hand, all-to-all topologies can have issues too like some network links may be faster than others (e.g. due to network congestion), with the result that some replication messages may "overtake" others. To order these events correctly, a technique called version vectors can be used.

### Leaderless Replication
Some data storage systems take a different approach, abandoning the concept of a leader and allowing any replica to directly accept writes from clients.

In some leaderless implementations, the client directly sends its writes to several replicas, while in others, a coordinator node does this on behalf of the client. The coordinator does not enforce a particular ordering of writes.

## Summary

### Replication can serve several purposes:

* High availabilty: Keeping the system running, even when one machine (or several machines, or an entire datacenter) goes down.

* Disconnected operation: Allowing an application to continue working when there is a network interruption

* Latency: Placing data geographically close to users, so that users can interact with it faster

* Scalability: Being able to handle a higher volume of reads that a single machine could handle, by performing reads on replicas.

### There are three main approaches to replication:
* Single leader replication: Clients send all writes to a single node (the leader), which sends a stream of data change events to the other replicas (followers). Reads can be performed on any replica, but reads from followers might be stale.

* Multi leader replication: Clients send each write to one of several leader nodes, any of which can accept writes. The leaders send streams of data change events to each other and to any follower nodes.

* Leaderless replicaton: Clients send each write to several nodes, and read from several nodes in parallel in order to detect and correct nodes with stale data.

### Deciding how an application should behave under replication lag:
* Read-after-write consistency: Users should always see data that they submitted themselves.

* Monotonic reads: After users have seen the data at one point in time, they shouldn't later see the data from some earlier point in time

* Consistent perfix reads: Users shoudll see the data in a state that makes causal sense: for example, seeing a question and its reply in the correct order.






