### Overview
----

In this section, we illustrate the smarter part of the network, the part that leverages the overlay functionalities to serve the other modules and ultimately the chain.

In specific terms, this section deals with communication models (_or specialized communication scenarios_) and the awareness models the network can make use of (_aware as in aware of the state it needs to perform this specialized communication_) and subsequently the architectural decisions they might engender.

### What is Peer Logic
---
Peer Logic is a everything peer specific in the context of an application that is a decentralized peer to peer one. Meaning, whatever behavior is expected to be performed from the perspective of a single peer in relation to others, specifically:

1. Communication models and protocols
2. Stateful peers/neighbors selection, ranking and ordering
3. Peer roles

### Specification
---

First, we need to discuss a few things a **_peer needs to do in the network_**.

We are interested in the two main state-touching scenarios:

1. Proposing

It has become apparent to us early on that the other layers (_like consensus for instance_) might have specific communication requirements --_either for security or performance purposes_, 

For instance, in Consensus, the proposer is required to directly communicate with the validators. Thus, it is incumbent on the peer-to-peer layer as a module to offer such functionality while _**also factoring it into the sum of all parts**_.

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
2. The edge peers are not required to be direct neighbors of the peer (_i.e: in the routing table of the nucleus peer_). 
3. The nucleus peer sequentially and linearly communicates with the nodes in the provided edge peers addresses list [a]
4. Order is irrelevant in this equation
5. Direct communication will have a separate channel(s) and will not be bound to maximum connections and connection pooling constraints. (_refer to the [Transport Protocol & Security Section](https://github.com/pokt-network/gemelos/wiki/Transport-Logic-And-Security) for more information_)
6. Direct communication messages will specify the `Direct` communication type in the messages header. (_Check [Messages In The Overlay](https://github.com/pokt-network/gemelos/wiki/Messages-In-The-Overlay) section for more information_)

[a]: Better ways of establishing direct communication from a singular nucleus towards a multitude of edges are also feasible, such as broadcast and multicast, however, for security and performance purposes, we will not cover this in this specification. In case there is a serious interest in exploring such avenues, a broadcast model could be possible if some underlying implementation allows it over the internet, say sockets for instance or some special router nodes to act as channels for such broadcast.

#### 2. Passaround Communication / Progressive Multicast

##### Description

A Passaround Communication model is a progressive multicast happening and propagating from peer to peer until the entire network is covered.

This model is similar to the direct communication model in as far as receiving acknowledgements of delivery is required, however is different in the fact that multi-unicasts don't stop at one star pattern but span over every other similar tree/star in the network, gaining the multicast property.

A visual illustration of this sequential process is present here and is as follow:

**Passaround initiated by a head node**
![Passaround as is from head node](https://i.ibb.co/1MxMyCm/Screen-Shot-2021-10-05-at-18-09-37.png)

**Passaround initiated by a child node**
![Passaround as in from child node](https://i.ibb.co/9bsbXwN/Screen-Shot-2021-10-05-at-18-11-45.png)

If we are to aggregate all peers and their rings in one network ring, it would be very similar to chord and as follows:

![Passaround Communication as in Chord](https://www.researchgate.net/profile/Mario-Kolberg/publication/262398264/figure/fig1/AS:669953283862535@1536740718640/An-example-Chord-network-showing-the-choice-of-finger-nodes-for-Node-N8_Q320.jpg)

Except that the jump will not be logarithmic but constant.

(_Refer to [Routing Structure And Algorithm](https://github.com/pokt-network/gemelos/wiki/Routing-Structure-And-Algorithm) section to learn more about the ring structure and how can this communication model be achieved_)

##### Formalization

The Passaround Communication model can be specified as follows:

A peer in a [Gemini](#) overlay is organized and ordered in within a number of affinity groups and a number of pointer groups.

A peer Y is concerned with passing around a message to its affinity group and delegating this task to other affinity groups so that the entire network receives this message.

To achieve this, we split every affinity group into k intervals (_with k=√n, and n peers in the group as best parameter_), and designate the first peer in each of those intervals as the interval's head.

From then on, a message is multicasted within an affinity group by first disseminating this message amongst interval heads in its group, each head goes on to send this message to each child it is responsible for. To ensure that this message is properly disseminated to the entire network, one of the interval heads makes sure to also delegate this message to other affinity groups through its pointers groups.

For a given peer Y to initiate a **Passing Round**, it performs the following:

1. Peer Y sends a [**Passaround Message**]() to its immediate head node in its affinity group(s)

     1.1. Peer Y must receive an acknowledgment of receipt from its head node.

     1.b. In case the immediate **head** of peer Y fails to acknowledge receipt within a **retry** window, peer Y marks this peer as a **skipped** peer in the message's header and passes it to successor of the failed peer as the new **head**.

     1.c. When a **head** peer receives a [**Passaround Message**](), it acknowledges the receipt of the message and passes the message to the other heads in the interval successor in the same fashion.

    1.d If a **head** peer fails to acknowledge the receipt of a message, its successor is taken as a new **head**.

     1.e If peer Y is a **head**, then peer Y is its own immediate head.

2. The head node of Peer Y then sends a [**Delegated Passaround Message**]() to all different affinity group(s) that exist in its pointers group by picking peers with unique prefixes (_within the same dimension_) [a], marking each peer as the **Next Head**.

   2.a. If a peer in the pointer group(s) fails to acknowledge receipt of the [**Delegated Passaround Message**](), peer Y will try to send the same message to a different peer that has the same affinity group as the failed one. If none found, that peer is simply skipped in that **Delegation round**.

   2.b. A peer receiving a **Delegated Passaround Message** will simply initiate a **Passing Round** in its own affinity group(s) as described in step 1.

   2.c. A **head** peer will try to delegate this message to the other non-covered affinity groups through its own affinity group(s) by hand-picking a peer that has a different pointer groups (_suffix_) and sending a [**Forwarded Delegated Passaround Message**] and marking it as a **Next Delegator**

   2.d. A peer receiving a **Forwarded Delegated Passaround Message** will not initiate a **Passing Around** in its affinity group(s), but will try to initiate a **Delegation Round** to other affinity groups through its pointers group(s) that were not covered yet. In case it fails, it will try to initiate a **_Delegation Forwarding Round_** in the same way described in step (2.c)

3. If a peer receiving a **Passaround Message** or **Delegated Passaround Message** or **Forwarded Delegated Passaround message** has seen this message **MaxSeenTimes**, it will not pass it around nor delegate it.
  
4. The next delegator will also pick a new next delegator, until no next delegator is able to find new affinity groups to delegate to.
5. The next delegator can ignore the fact that everybody has already received (_if it runs out of new affinity groups to route to_) the message and go for for extra delegation rounds if **_MaxRedundancy_** is not reached yet.


The Passaround Algorithm is as follows:

![Passaround Algorithm 1 in Latex](https://i.ibb.co/aaBjQ3MDc/Screen-Shot-2021-09-29-at-03-05x-53.png)

![Passaround Algorithm 2 in Latex](https://i.ibb.co/aaqncNPVm/Screen-Shot-2021-09-27-at-13-21x-54.png)

![Passaround Algorithm 3 in Latex](https://i.ibb.co/aatJBY7Zx/Screen-Shot-2021-09-29-at-03-06x-08.png)

![Passaround Algorithm 4 in Latex](https://i.ibb.co/aaPDVsz5S/Screen-Shot-2021-09-27-at-13-22x-09.png)

As copiable MD version is as follows
```

```

This algorithm should is given as an example for a gemini dimension equals `gd=1`.

To extend it to `gd=n`, simply Pass, Delegate and Forward Delegation in each dimension's Affinity group, and increase **MaxSeenTimes** to account for the number of dimensions.

**Redundancy estimation and cost**:

In the current way this is setup, a given affinity group will not receive a message more than once due to the mere fact that we explicitly track impacted affinity group to not send them messages again.

To achieve some level of redundancy, we will simply go for a new round of multicast for a given message, once the first round finishes, and to achieve this, we will simply reset the impact affinity groups tracker as if we have not reached any affinity group yet and basically restart the entire round from scratch.

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

The Chain-aware leader indexes peer addresses by the height they are at. To avoid indexing the entire network, the Chain-aware leader indexes its entire group per heights, and keeps an index of other groups however with only 3 addresses per height.

The Chain-aware leader keeps this index up to date by:

1. Relying on peers in its affinity group to push to their heights.
2. Relying on other Chain-aware leaders from other groups to exchange their groups indexes.

_Read the next chapter for information_

A Chain-aware leader will only keep a certain range of heights in its awareness-index.

**1.2 Push-based awareness-state communication**

Every time a peer reaches a given height, it will inform the **_Chain-Aware Leader_** of its group and the **_Chain-Aware Replicas** as well. This approach is very straightforward and requires very little overhead in terms of management.

**_Chain Aware Leaders_** ensure that they are all up to date by forwarding [**_Chain-Awareness Sync Request_**](https://github.com/pokt-network/hydrate/wiki/Messages-In-The-Overlay) to other leaders in other groups using the **Targeted Passaround** communication model, which every leader considers by keeping only the updated heights received in this request.

**1.3 Chain-Awareness Model**

The **_Chain-Aware Leader_** and **_Chain-Aware Replicas_** will maintain a simple chain-awareness structure primarily ordered by a key chain state data point, in our case, chain height.

We would like to keep the data structure choice undecided as to keep innovation space for incoming implementations, however we would like to constraint such structure with the following requirements:

1. Read time should be constant: _O(1)_
2. Write time should be at worst logarithmic: _O(log(N)_
3. Entries should be ordered by a key chain state data point. (_aka height or other_)

This structure will be heavily read from by group peers that would like to learn about what peers to sync from.

The write on the other hand, or Chain-Awareness sync will be periodic and is accepting of a relatively _"worse"_ complexity.

#### 2. Reputation Aware Network (Blacklisted/Banned peers)
// TBD

#### 3. Geography Aware Network
// TBD

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
// Express a full scenario of how the direct communication model will be used.
#### 2. Syncing
// Express a full scenario of how a peer will talk to its affinity group(s) chain-aware leader and request sync information for specific heights and what happens the chain-aware leader is unable to do so. (_We will technically fallback on a passaround message to ask everybody for that specific height, and everyone with that height will use Targeted Passaround to send their address to the requesting node for a direct sync to start_)