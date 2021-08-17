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