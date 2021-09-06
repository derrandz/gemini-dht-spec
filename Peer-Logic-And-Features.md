## Overview
----

In this section, we illustrate the smarter part of the network, the part that leverages the overlay functionalities to serve the other modules and ultimately the chain.

In specific terms, this section deals with communication models (_or specialized communication scenarios_) and the awareness models the network can make use of (_aware as in aware of the state it needs to perform this specialized communication_) and subsequently the architectural decisions they might engender.

## Peer Logic
----
First, we need to discuss a few things a **_peer needs to do in the network_**. We are interested in the two main state-touching scenarios:

1. Proposing

It has become apparent to us early on that the consensus layer establishes a communication requirement to meet its security requirements, namely requiring the proposer to directly communicate with the concerned validators. Thus, it is incumbent on the peer-to-peer layer as a module to offer such functionality while _**also factoring it into the sum of all parts**_.

2. Syncing

When a node is trying to sync the blockchain, we would like the network to be smart enough to know what other peers are the best candidates to sync from and also do so rather efficiently. Since discovering such network state can be expensive to do in a cold fashion (_ specially at scale_), we hope to describe methods by which the peer to peer module will ensure maximum efficiency in doing so.

## Features
----

### Specialized Communications (_Communication models_)
---

#### 1. Direct/STAR Communication
#### 2. Trickled Multicast
#### 3. Eventual Propagation

### Awareness models (_Specialized states_)
---

#### 1. Chain Aware Network
#### 2. Geography Aware Network

