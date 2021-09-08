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

When a node is trying to sync the blockchain, we would like the network to be smart enough to know what other peers are the best candidates to sync from and also do so rather efficiently. Since discovering such network state can be expensive to do in a cold fashion (_specially at scale_), we hope to describe methods by which the peer to peer module will ensure maximum efficiency in doing so.

## Features

### Specialized Communications (_Communication models_)
---

#### 1. Direct/STAR Communication

##### Description
The star communication model is as the name implies, a one central nucleus who is trying to communicate with a multitude of edges.

A visual illustration is present here as follow:

![Hotstuff's star communication pattern](https://miro.medium.com/max/1136/1*Ag8SrZlFbMUQZAZR9n0r0A.png)

The goal of such communication model is to directly communicate with a list of peers while maintaining connection with them to perform whatever acknowledgment behavior or request/response behavior.

At this point, we envisage that this feature will be primarily used by the proposer-to-validators relationship, but could potentially used in the future for other purposes.

##### Formalization

The direct communication model is specified as follows:

1. The nucleus peer expects to have its edge peers addresses in a list, either produced from its routing table or retrieved from the chain or some other party*.
2. The edge peers are not required to be in the routing table of the nucleus peer. 
3. The nucleus peer sequentially and linearly communicates with the nodes in the provided edge peers addresses list
4. Order is irrelevant in this equation
5. Direct communication will have a separate channel(s) and will not be bound to maximum connections and connection pooling constraints. (_refer to the [Transport Protocol & Security Section](https://github.com/pokt-network/gemelos/wiki/Transport-Protocols-And-Security) for more informtation_)
6. Direct communication messages will specify the `Direct` communication type in the messages header. (_Check [Messages In The Overlay](https://github.com/pokt-network/gemelos/wiki/Messages-In-The-Overlay)_)

#### 2. Trickled Multicast

![Circular trickle down communication as in Chord](https://www.researchgate.net/profile/Mario-Kolberg/publication/262398264/figure/fig1/AS:669953283862535@1536740718640/An-example-Chord-network-showing-the-choice-of-finger-nodes-for-Node-N8_Q320.jpg)

#### 3. Eventual Propagation
![Tree-based multicast](https://i.ibb.co/f2gpg9z/Screen-Shot-2021-09-07-at-12-37-19-AM.png)


### Awareness models (_Specialized states_)
---

#### 1. Chain Aware Network
#### 2. Geography Aware Network
#### 3. Reputation Aware Network (Blacklisting and stuff)

### Application
---

#### 1. Proposing
#### 2. Syncing