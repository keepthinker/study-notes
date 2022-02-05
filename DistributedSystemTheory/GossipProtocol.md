Expressing these ideas in more technical terms, a gossip protocol is one that satisfies the following conditions:

- The core of the protocol involves periodic, pairwise, inter-process interactions.

- The information exchanged during these interactions is of bounded size.

- When agents interact, the state of at least one agent changes to reflect the state of the other.

- Reliable communication is not assumed.

- The frequency of the interactions is low compared to typical message latencies so that the protocol costs are negligible.

- There is some form of randomness in the peer selection. Peers might be selected from the full set of nodes or from a smaller set of neighbors.

- Due to the replication there is an implicit redundancy of the delivered information.