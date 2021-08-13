# Chapter 5: Replication

*Replication* means keeping a copy of the same data on multiple machines that are connected via a network.  There are several reasons why'd we do this including:

* To keep data geographically close to your users (and thus reduce latency)
* To allow the system to continue working even if some of its parts have failed (and thus increase availability)
* To scale out the number of machines that can serve read queries (and thus increase read throughput)

Replicaion lies in handling changes to repicated data (compared to a static dataset). There are three popular algorithms for replicating changes between nodes: single-leader, multi-leader, and leaderless.