# The properties of consensus algorithms
- They ensure safety (never returning an incorrect result) under all non-Byzantine conditions,
including network delays, partitions, and packet loss, duplication, and reordering.

- They are fully functional (available) as long as any majority of the servers are operational
and can communicate with each other and with clients. Thus, a typical cluster of five servers
can tolerate the failure of any two servers. Servers are assumed to fail by stopping; they may
later recover from state on stable storage and rejoin the cluster.

- They do not depend on timing to ensure the consistency of the logs: faulty clocks and extreme message delays can, at worst, cause availability problems. That is, they maintain safety under an asynchronous model, in which messages and processors proceed at arbitrary speeds.
- In the common case, a command can complete as soon as a majority of the cluster has responded to a single round of remote procedure calls; a minority of slow servers need not impact overall system performance.

# Raft overview
- **Leader election**: a new leader must be chosen when starting the cluster and when an existing leader fails.
- **Log replication**: the leader must accept log entries from clients and replicate them across the cluster, forcing the other logs to agree with its own.
- **Safety**: the key safety property for Raft is the State Machine Safety Property: if any server has applied a particular log entry to its state machine, then no other server may apply a different command for the same log index. 


# Raft Basics
A Raft cluster contains several servers, at any given time each server is in one of three states: leader, follower, or candidate. In normal operation there is exactly one **leader** and all of the other servers are **followers**. Followers are passive: they issue no requests on their own but simply respond to requests from leaders and candidates. The leader handles all client requests (if a client contacts a follower, the follower redirects it to the leader). The third state, **candidate**, is used to elect a new leader.

Raft divides time into terms of arbitrary length. Terms are numbered with consecutive integers. Each term begins with an election.

Current terms are exchanged whenever servers communicate; if one server's current term is smaller than the other's, then it updates
its current term to the larger value. 

# Leader election
A candidate wins an election if it receives votes from a majority of the servers in the full cluster for the same term. The majority rule ensures that at most one candidate can win the election for a particular term.

A candidate continues in this state until one of three things happens: 

## It wins the election
A candidate wins an election if it receives votes from a majority of the servers in the full cluster for the same term. Each server will vote for at most one candidate in a given term, on a firstcome-first-served basis.

## Another server establishes itself as leader.
While waiting for votes, a candidate may receive an AppendEntries RPC from another server claiming to be leader. If the leader's term (included in its RPC) is at least as large as the candidate's current term, then the candidate recognizes the leader as legitimate and returns to follower state. If
the term in the RPC is smaller than the candidate's current term, then the candidate rejects the RPC and continues in candidate state.

## Another election timeout goes by with no winner.
If many followers become candidates at the same time, votes could be split so that no candidate obtains a majority. When this happens, each candidate will time out and start a new election by incrementing
its term and initiating another round of RequestVote RPCs.


# Log replication
Each client request contains a
command to be executed by the replicated state machine. The leader appends the command to its log as a new entry, then issues AppendEntries RPCs in parallel to each of the other servers to replicate the entry. 

A log entry is committed once the leader that created the entry has replicated it on a majority of the servers.

The leader decides when it is safe to apply a log entry to the state machines; such an entry is called committed. Raft guarantees that committed entries are durable and will eventually be executed by all of the available state machines.

Raft maintains the following properties, which
together constitute the Log Matching Property:
- If two entries in different logs have the same index and term, then they store the same command.
- If two entries in different logs have the same index and term, then the logs are identical in all preceding entries.

# Safty
## Election Restriction
In any leader-based consensus algorithm, the leader must eventually store all of the committed log entries. 

Raft uses the voting process to prevent a candidate from winning an election unless its log contains all committed entries.

Raft determines which of two logs is more up-to-date by comparing the index and term of the last entries in the logs. If the logs have last entries with different terms, then the log with the later term is more up-to-date. If the logs end with the same term, then whichever log is longer is more up-to-date.

## Committing entries from previous terms
Raft never commits log entries from previous terms by counting replicas. Only log entries from the leader's current term are committed by counting replicas; once an entry from the current term has been committed in this way, then all prior entries are committed indirectly because of the Log Matching Property.
![image](https://i.stack.imgur.com/xHgAH.png)

