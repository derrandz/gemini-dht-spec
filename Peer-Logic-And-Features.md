## Overview
----

In this section, we illustrate the smarter part of the network, the part that leverages the overlay functionalities to serve the other modules and ultimately the chain.

In specific terms, this section deals with communication models (_or specialized communication scenarios_) and the awareness models the network can make use of (_aware as in aware of the state it needs to perform this specialized communication_) and subsequently the architectural decisions they might engender.

## What is Peer Logic
---



## Specification
---

First, we need to discuss a few things a **_peer needs to do in the network_**. We are interested in the two main state-touching scenarios:

1. Proposing

It has become apparent to us early on that the consensus layer establishes a communication requirement to meet its security requirements, namely requiring the proposer to directly communicate with the concerned validators. Thus, it is incumbent on the peer-to-peer layer as a module to offer such functionality while _**also factoring it into the sum of all parts**_.

2. Syncing

When a node is trying to sync the blockchain, we would like the network to be smart enough to know what other peers are the best candidates to sync from and also do so rather efficiently. Since discovering such network state can be expensive to do in a cold fashion (_specially at scale_), we hope to describe methods by which the peer to peer module will ensure maximum efficiency in doing so.

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

#### 2. (WIP: will need to incorporate network phase) Passaround Communication / Progressive Multicast

##### Description

A Passaround Communication model is a progressive multicast happening and propagating from peer to peer until the entire network is covered.

This model is similar to the direct communication model since the peer who initiates the trickling (_which shall be referred to from now on as the **head** peer_) requires to receive a final acknowledge of the delivery of the message to all the other peers, but is different in the fact that it only tells a next node about this whole trickling around.

The concept is nothing new apart from the fact that we picked for it an expressive (_and rather unambiguous_) name, and is already present in (_for example_) Chord.

A visual illustration of this sequential process is present here and is as follow:

![Passaround as is](https://i.ibb.co/kQJ7yT9/Trickel-Around.png)

If we are to aggregate all peers and their rings in one network ring, it would be very similar to chord and as follows:

![Passaround Communication as in Chord](https://www.researchgate.net/profile/Mario-Kolberg/publication/262398264/figure/fig1/AS:669953283862535@1536740718640/An-example-Chord-network-showing-the-choice-of-finger-nodes-for-Node-N8_Q320.jpg)

Except that the jump will not be logarithmic but sequential and also semi-random.

(_Refer to [Routing Structure And Algorithm](https://github.com/pokt-network/gemelos/wiki/Routing-Structure-And-Algorithm) section to learn more about the ring structure and how can this trickling be achieved_)

##### Formalization

The Passaround Communication model can be specified as follows:

1. The `Head` peer sends a Passaround Message to the next three "random" nodes in its hat club.
2. The `Head` peer records its hat club in the Passaround Message and sends it to a random node with a different hat club in its boot club as the next `Head` (_by updating the `Head` value to match this random node's address_). 
3. Each node receiving a Passaround Message will first verify if it received it before, and then proceed to sending it to the next three peers in its hat club.
4. If a node has received a message `maxPasses` times, it does not forward it.
5. The first node in a new hat club to receive a Passaround Message with a `NextHead` equals to its address is then the `Head` peer of that hat club.
6. That next head goes on to repeat the logic of the `Head` of a club and Passes ARound that given message.

The Passaround Algorithm is as follows:
```
Passaround(M):
  P <- Current Peer
  
  if MessagesPool[M].Seen == 3:
     return

  A, B, C <- PickRandomNodes(P.HatClub)
  for E in [A, B, C]: P.Send(M, E)
  
  if M.Head == P.Address AND MessagesPool[M].Seen == 0:
    if M.CoveredHatClubs.Length >= HatClubsCount:
       return

    N <- PickRandomNodeWithDifferentHatClub(P.BootClub)

    M.Head <- N.Address
    M.CoveredHatClubs.Push(P.HatCase)

    P.Send(M, N)

  MessagesMap[M].Seen++
  
```

#### 3. (WIP) Targeted Passaround Communication / Targeted Crawling Multicast

### Awareness models (_Specialized states_)
---

#### 1. Chain Aware Network

##### Description
In order to achieve network awareness (_learning more about the peers' states and organizing them according to it_), we opt to go with a leader-based approach to optimize in terms of bandwidth and complexity, as learning about neighboring node's states and propagating such information will be costly, and at all cases will be cheaper if taken care of by a singular node per group.


##### Formalization

**1. Chain Awareness**
 
Primarily, we are interested in other peers current heights, so that we facilitate processes such as blockchain syncing. For that, we will primarily implement:

A. A leader-based approach
B. A push-based communication model

**1.1 Chain-Aware Leader**

A Chain-aware leader of a group is basically the successor of the media point of a group. This leader is elected simply by convention and relies on the fact that all peers in a group are interconnected and have up-to-date routing states.

This leader is easily elected (_or rather is beforehand known by convention_) due to the cheer dumb luck of being the numerical center of its group (_address space interval_)

The three nodes to follow the leader, which we will denote **Chain-ware replicas** will gradually replicate the awareness state of the leader to ensure quick leaders replacement in case of outages.

The Chain-aware leader will act as a central entity for its group and provide requesting nodes with destination address for any height they request or would like to sync from.

**1.2 Push-based awareness-state communication**

Every time a peer reaches a given height, it will inform the **_Chain-Aware Leader_** of its group and the **_Chain-Aware Replicas** as well. This approach is very straightforward and requires very little overhead in terms of management.

**_Chain Aware Leaders_** ensure that they are all up to date by forwarding [**_Chain-Awareness Sync Request_**](https://github.com/pokt-network/hydrate/wiki/Messages-In-The-Overlay) to other leaders in other groups using the **Targeted Passaround** communication model.

**1.3 Chain-Awareness Model**

The **_Chain-Aware Leader** and **_Chain-Aware Replicas_** will maintain a simple chain-awareness structure primarily ordered by a key chain state data point, in our case, chain height.

We would like to keep the data structure choice undecided as to keep innovation space for incoming implementations, however we would like to constraint such structure with the following requirements:

1. Read time should be constant: _O(1)_
2. Write time should be at worst logarithmic: _O(log(N)_
3. Entries should be ordered by a key chain state data point. (_aka height or other_)

This structure will be heavily read from by group peers that would like to learn about what peers to sync from.

The write on the other hand, or Chain-Awareness sync will be periodic and is accepting of a relatively _"worse"_ complexity.

#### 2. Geography Aware Network
#### 3. Reputation Aware Network (Blacklisting and stuff)

### Acting Models (_Specialized Peers/Peer Roles_)
---

#### Description

#### 1. (WIP) Seed Nodes

##### Description

The Seed role is a primary network role and has as a direct responsibility:

1. Keeping tracking of joining peers
1.a Keep track of their count
1.b Keep track of their addresses
1.c Keep periodically track of their aliveness but on a low frequency.

2. Provide joining peers with routing state bootstrap parameters
2.a Provide the [_Gemini dimension_](https://github.com/pokt-network/hydrate/wiki/Routing-Algorithm-And-Structure#521-exploiting-multi-dimensionality) in function of the network phase.
2.b Provide the [_Gemini Parameters_](https://github.com/pokt-network/hydrate/wiki/Routing-Algorithm-And-Structure#31-glossary) in function of the network phase
2.c After having decided on the parameters, provide the joining peer with at least on member per routing state dimension to ensure the continutation of the joining process (_member list exchange and membership_)

3. Trigger network phase change event when all conditions are met and coordinate peers routing state update in function of the phase.
// To be continued

#### 2. Validator Nodes
#### 3. Full nodes
#### 4. N/A

### Application
---

#### 1. Proposing
#### 2. Syncing