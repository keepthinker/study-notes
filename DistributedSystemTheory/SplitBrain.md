# Split-brain

It indicates data or availability inconsistencies originating from the maintenance of two separate data sets with overlap in scope, either because of servers in a [network design](https://en.wikipedia.org/wiki/Network_planning_and_design "Network planning and design"), or a failure condition based on servers not communicating and synchronizing their data to each other. This last case is also commonly referred to as a [network partition](https://en.wikipedia.org/wiki/Network_partition "Network partition").





## Approaches for dealing with split-brain

### The optimistic approaches

The optimistic approaches simply let the partitioned nodes work as usual; this provides a greater level of availability, at the cost of sacrificing correctness. Once the problem has ended, automatic or manual reconciliation might be required in order to have the cluster in a consistent state. One current implementation for this approach is [Hazelcast](https://en.wikipedia.org/wiki/Hazelcast "Hazelcast"), which does automatic reconciliation of its key-value store.



## The pessimistic approaches

The pessimistic approaches sacrifice availability in exchange for consistency. Once a network partitioning has been detected, access to the sub-partitions is limited in order to guarantee consistency. A typical approach, as described by Coulouris et al., is to use a [quorum](https://en.wikipedia.org/wiki/Quorum_(distributed_computing) "Quorum (distributed computing)")-consensus approach. This allows the sub-partition with a majority of the votes to remain available, while the remaining sub-partitions should fall down to an auto-[fencing](https://en.wikipedia.org/wiki/Fencing_(computing) "Fencing (computing)") mode. One current implementation for this approach is the one used by [MongoDB](https://en.wikipedia.org/wiki/MongoDB "MongoDB") replica sets. And another such implementation is Galera replication for [MariaDB](https://en.wikipedia.org/wiki/MariaDB "MariaDB") and [MySQL](https://en.wikipedia.org/wiki/MySQL "MySQL").



Distributed systems don't typically suggest an odd number of nodes is what prevents split brain. Rather, it's majority *quorums* that avoid split brains. If a protocol chooses the node with a majority of votes to be the leader, and the protocol can ensure that nodes will only ever choose one leader, then logically there can only be one because there can only be one majority. Often, odd numbers of nodes are only used because they provide the greatest level of fault tolerance, e.g. a majority of 2 is 2, but a majority of 3 is also 2. That gives you room to tolerate one failure while still being able to acquire a majority of votes for a leader.

Reference: https://stackoverflow.com/questions/46107962/how-does-an-odd-number-solve-a-split-brain-in-a-distributed-system


