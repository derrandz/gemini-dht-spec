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

A visual illustration is present here and is as follow:

![Hotstuff's star communication pattern](https://miro.medium.com/max/1136/1*Ag8SrZlFbMUQZAZR9n0r0A.png)

The goal of such communication model is to directly communicate with a list of peers while maintaining connection with them to perform whatever acknowledgment behavior or request/response behavior.

At this point, we envisage that this feature will be primarily used by the proposer-to-validators relationship, but could potentially used in the future for other purposes.

##### Formalization

The direct communication model is specified as follows:

1. The nucleus peer expects to have its edge peers addresses in a list, either produced from its routing table or retrieved from the chain or some other party*.
2. The edge peers are not required to be in the routing table of the nucleus peer. 
3. The nucleus peer sequentially and linearly communicates with the nodes in the provided edge peers addresses list
4. Order is irrelevant in this equation
5. Direct communication will have a separate channel(s) and will not be bound to maximum connections and connection pooling constraints. (_refer to the [Transport Protocol & Security Section](https://github.com/pokt-network/gemelos/wiki/Transport-Logic-And-Security) for more information_)
6. Direct communication messages will specify the `Direct` communication type in the messages header. (_Check [Messages In The Overlay](https://github.com/pokt-network/gemelos/wiki/Messages-In-The-Overlay) section for more information_)

#### 2. Trickled Around Communication / Progressive Multicast

##### Description

A Trickled Around Communication model is a progressive multicast happening and propagating from peer to peer until the entire network is covered.

This model is similar to the direct communication model since the peer who initiates the trickling (_which shall be referred to from now on as the **head** peer_) requires to receive a final acknowledge of the delivery of the message to all the other peers, but is different in the fact that it only tells a next node about this whole trickling around.

The concept is nothing new apart from the fact that we picked for it an expressive (_and rather unambiguous_) name, and is already present in (_for example_) Chord.

A visual illustration of this sequential process is present here and is as follow:

![Trickled Around as is](https://i.ibb.co/kQJ7yT9/Trickel-Around.png)

If we are to aggregate all peers and their rings in one network ring, it would be very similar to chord and as follows:

![Trickled Around Communication as in Chord](https://www.researchgate.net/profile/Mario-Kolberg/publication/262398264/figure/fig1/AS:669953283862535@1536740718640/An-example-Chord-network-showing-the-choice-of-finger-nodes-for-Node-N8_Q320.jpg)

Except that the jump will not be logarithmic but sequential and also semi-random.

(_Refer to [Routing Structure And Algorithm](https://github.com/pokt-network/gemelos/wiki/Routing-Structure-And-Algorithm) section to learn more about the ring structure and how can this trickling be achieved_)

##### Formalization

The Trickled Around Communication model can be specified as follows:

1. The `Head` peer sends a Trickle Around Message to the next three "random" nodes in its hat club.
2. The `Head` peer records its hat club in the Trickle Around Message and sends it to a random node with a different hat club in its boot club as the next `Head` (_by updating the `Head` value to match this random node's address_). 
3. Each node receiving a Trickle Around Message will first verify if it received it before, and then proceed to sending it to the next three peers in its hat club.
4. If a node has received a message `maxTrickles` times, it does not forward it.
5. The first node in a new hat club to receive a Trickle Around Message with a `NextHead` equals to its address is then the `Head` peer of that hat club.
6. That next head goes on to repeat the logic of the `Head` of a club and Trickles Around that given message.

The Trickle Around Algorithm is as follows:
```
TrickleAround(M):
  P <- Current Peer
  
  if MessagesPool[M].Seen == 3:
     return

  A, B, C <- PickRandomNodes(P.HatClub)
  for E in [A, B, C]: P.Send(M, E)
  
  if M.Head == P.Address AND MessagesPool[M].Seen == 0:
    N <- PickRandomNodeWithDifferentHatClub(P.BootClub)

    M.Head <- N.Address
    M.CoveredHatClubs.Push(P.HatCase)

    P.Send(M, N)

  MessagesMap[M].Seen++
  
```


### Awareness models (_Specialized states_)
---

#### 1. Chain Aware Network
#### 2. Geography Aware Network
#### 3. Reputation Aware Network (Blacklisting and stuff)

### Acting Models (_Specialized Peers/Peer Roles_)
---

#### 1. Seed Nodes
#### 2. Validator Nodes
#### 3. Full nodes
#### 4. N/A

### Application
---

#### 1. Proposing
#### 2. Syncing