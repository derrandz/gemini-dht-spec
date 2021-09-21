## I. Overview
----
Choosing the proper data structure to represent the structure of a network's overlay is the main and the crucial step to achieving a structured overlay, and a detrimental one for building an efficient and performant network.

In this section, we go try to explain why we chose Gemini as the main structure and algorithm as well as describe a spec for implementation that we hope is going to serve as a reference.

Before we dive in this section, we want to allude to the fact that the paper itself lacks any reference implementation, and is not very rigorous in defining Mathematical terms (_We've had access to three different versions_), so we will provide our own answers in multiple cases.

**A reminder of our visual toolbox** (_while thanking devp2p spec for the inspiration_):


💡: indicates an implementation avenue or suggestion

✍🏻: A new term we chose to use for convenience purposes

🗝: Key concept that should be paid attention to

## II. The Problematic
----

During our [research](#), we have been able to identify the most suitable candidate for our routing algorithm and overlay structure, which ended up being Gemini (_more on this down below_), however, having ran a few simulations and performed a few projections, we realized that as scalable as Gemini is, we could still face some challenges before we reach a relatively significant network size (_peer count_)

All of a sudden, the challenge was not scaling to a billion nodes but rather accommodating for cases when the network is still at its infancy and growing, a 1000 and below.

We first off tried to keep it as simple as possible and tried to go with an if else approach. If below a certain size, use a PRR algorithm, if beyond the threshold, fallback to Gemini.

However this approach would have been a hell to implement, due to the fact that past the defined threshold, you would effectively almost join a different network, with a different churn management process and different discovery logic and what have you. No one was willing to implement that.

After some thought, and after revisiting other candidates we have covered in our research, we took a special appeal to Kelips O(1) routing algorithm, but we hoped to extract the good parts only, so that we don't have to deal with the file-lookup specific stuff and what not.

We were pretty sure that Gemini was the best candidate for our network regardless of what size it is, since it is highly parameterized, however we wanted an efficient network at whatever size.

To make sure that we explain everything pretty well, this document will embody the decision making timeline, by first:

 1. Explaining why Gemini
 2. What are the challenges of using Gemini before reaching a significant network
 3. The solution (_A hybrid appraoch_)

## III. Specification
---

### 1. What is Gemini?

Gemini is a structure with an algorithm aimed to support efficient and scalable structured overlays.
Its name comes from the fact that its routing table consists of two parts, one containing nodes with common prefix and the other containing nodes with common suffix. Gemini routes messages to their destinations in just 2 hops, with a very high probability.
Although Gemini is not formally a O(1) routing algorithm, however it can be categorized as so. The same can be said about it if we want to categorize it as a logarithmic degree mesh or a PRR algorithm (_in terms of algorithm similarity, obviously the asymptotic complexity is significantly enhanced in the case of Gemini_)

In our opinion, it has gotten the best of all worlds.

You can find more information about Gemini in the following papers:

  1. [Gemini Paper first version](https://link.springer.com/content/pdf/10.1007%2F978-3-540-24685-5_22.pdf)
  2. [Gemini Paper second version](https://www.researchgate.net/publication/221601988_Gemini_Probabilistic_Routing_Algorithm_in_Structured_P2P_Overlay/link/53fdb64c0cf22f21c2f82a31/download)
  3. [Kelips Paper](http://iptps03.cs.berkeley.edu/final-papers/kelips.pdf)

### 2. Why Gemini?

 
1) Gemini divides peers into a symmetric manner, allowing for maximum control and configuration.
2) Gemini nodes employ items in the other dimension of groups as pointers allowing peers to have a reduced fixed scope with high "dispersed-ness", which means most of the network will be covered in that small scope. (smaller routing table).
3) In Gemini whether a message can successfully reach its target in one hop is in some probability. Reducing this probability can trade hop count for bandwidth overhead, making Gemini more scalable than others.
4) Unlike other overlays that are designed for file lookups, Gemini is just a substrate, making it lighter and leaving data operations to its application designers, and making it an ideal candidate for a role such as "Network backbone".
5) Gemini is simple, no specifically complex classification or routing logic is required, Gemini relies on natural uniformity and distribution laws that are observably always true as long as parameters are respected, which significantly reduces human-error chances.
6) No better candidate! You can refer to the our research summary to learn more about this. // TODO: Link to Forum summarizing our research effort or add in as a separate page in the wiki


### How does Gemini work in short?

// TBD 

### 3. Gemini Specification
----

#### 3.1 Glossary

In this section, we will start by defining the main terms used by the paper to remove any ambiguity. Those will be used heavily in this specification and we hope in any implementing code.

* **Hat case**: first `h` bits of a node's ID
* **Boot case**: last `b` bits of a node's ID
* **Node id**: an ID resulting from a _uniformly distributed hash function*_ 
* **Numerically close**: A random node A is said to be the numerically closest to node B if: _for every node X out of node A's neighbors `|IDa - IDb| < |IDa - IDx|`_
* **Numerically closest** (🗝): A random node A's ID is the numerically closest node to a node B if: _For every node X in node B's neighbors_: `min(|IDa - IDx|) => X = B`. We can say that node A is the numerically closest node to itself since: `|IDa - IDa| = 0 and 0 < |IDa - IDx|`
* **Hat club**: The set of nodes sharing the same first `h` bits of their IDs
* **Boot club**: The set of nodes sharing the same last `b` bits of their IDs
* **Root node / Destination node**: The destination node of a Message M is the node with the ID which is the _**numerically closest**_ ID to M's destination ID. (_please make the distinction between destination node and destination ID_)
* **Head of club** (✍🏻): Is the concerned node in a hat club. If we are routing from node X, node X is then said to be the head of its own Hat and Boot Clubs.

> To learn more about the nature and structure of the IDs used with this algorithm, please refer to [Node Identification And Security](https://github.com/pokt-network/gemelos/wiki/Node-Identification-And-Security)

#### 3.2 Routing Data Structure

The routing data structure is comprised of a hat club and a boot club representation of the network relative to the concerned node, and a formal way of describing it would be:

```
Rt(A) = { HatClub(A) U BootClub(A) }
```

Meaning, the routing table of a node A is the union of its hat club and boot club.

Let's start by describing the hat and boot clubs and how they make up a routing table.

##### Hat Club

We define `n` as the length of IDs assigned to node, A hat club is represented as an address space (N>>2^n) such that each ID belonging to a hat club shares the same `h` bits with the concerned node's ID.

In a given hat club:

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to the head's ID (representing "the start point on the ring").
  * IDs partaking in a hat club are all unique.
  * A hat club can measure the distance between a given ID and its head's ID.
  * A hat club can calculate the **numerically closest* ID(s) to its head.

##### Boot Club

We define `n` as the length of IDs assigned to node, A boot club is represented as an address space (N>>2^n) such that each ID belonging to a boot club shares the same `b` bits with the concerned node's ID.

In a given boot club:

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to center's ID (representing "the start point on the ring").
  * IDs partaking in a boot club are all unique.
  * A boot club can measure the distance between a given ID and its head's ID.
  * A boot club can calculate the **numerically closest* ID to its head.

#### 3.3 Routing Algorithm


To route a given message M, we define the following properties of the message that will be useful to the routing process:

  * A message has a n bits long destination ID
  * A message has a n bits long sender ID

(_To learn more about messages, please refer to [Messages In The Overlay](https://github.com/pokt-network/hydrate/wiki/Messages-In-The-Overlay)_)

To route a message M, node N does the following:

  1. Verify if the destination ID of the message M belongs to N's hat club.
     1.a If M's destination ID belongs to N's hat club, N routes the message to the **numerically closest** ID to the M's destination ID
     1.b If not, N looks for a node E in its boot club such that E's ID is in the same hat club as in M's destination ID.
       1.b.a If found, node N routes the message M to node E
       1.b.b If not, node N routes E to a randomly picked node from its hat case such that E's ID has a different boot case than M's destination ID.

Written in pseudo-code, the Gemini routing algorithm is as follows:
![Gemini Routing Algorithm Written In Pseudocode](https://i.ibb.co/m5hysTY/Screen-Shot-2021-09-15-at-4-27-31-AM.png)

Here is a copiable version in Markdown

```
route(M):
  destID <- M's destination ID
  if destID belongs to HatClub(N)
  then:
     Route to the numerically closest ID to destID;
     return;
  else:
    for every E in BootClub(N):
      if destID belongs to HatClub(E)
      then:
         Route to E;
         return;
 
  Found, E, Attempts <- False, Null, 0
  while !Found && Attempts < Length(HatClub(N):
    RandomNode <- PickRandomNode(HatClub(N)
    if not RandomNode belongs to BootClub(destID):
       E <- RandomNode
       Found <- true;
       return
    else
      Attempts++;

  Route to E.
```

#### 3.4 Maintenance

A peer cares to maintain only its reduced fixed scope of the network, that is its hat and boot clubs. To be able to maintain an updated view of the network, a Gemini peer sends periodic heartbeats to its clubs members and updates it state accordingly.

To see the full details of this process, please refer to the [Maintenance](https://github.com/pokt-network/hydrate/wiki/Churn-Management#4-maintenance) Section in [Churn Management](https://github.com/pokt-network/hydrate/wiki/Churn-Management) Chapter of this wiki/spec.

#### 3.5 Network Parameters and Scalability

So far, we've only talked about Gemini using abstract undefined parameters, specifically:

 1. `N`: Network size
 2. `n`: Peer IDs/Addresses Length
 3. `h`: Hat case length
 4. `b`: Boot case length

Without going into the full details of how these parameters influence the routing probability (_which is clearly and thoroughly available in the paper_), we will present our data findings as per our [simulation efforts](https://docs.google.com/spreadsheets/d/13nhNu94_lFnfuSl-4eonAGY72sX3n1bICKFyODkCTU0/edit?usp=sharing) to give you an idea about the scalability of the structure.

Before we start to present the data, we would like to remind you of the promise of Gemini, that is, if you set your parameters correctly, you will be able to perform routing in 2 hops 99% of the time.

##### Optimal parameters for a 6000 peers network:

| Parameter                        |        |
|----------------------------------|--------|
| N                                |  6000  |
| h                                |   5    |
| b                                |   3    |
| H                                |   32   |
| B                                |   8    |
| Probability of Routing in 2 hops |   96%  |
| Routing State Size               |   937  |

##### Optimal theoretical parameters for a 10K peers network:


| Parameter                        |         |
|----------------------------------|---------|
| N                                |  10000  |
| h                                |   4     |
| b                                |   4     |
| H                                |   16    |
| B                                |   16    |
| Probability of Routing in 2 hops |   97%   |
| Routing State Size               |   1250  |
 


##### Optimal theoretical parameters for a 100K peers network:

| Parameter                        |          |
|----------------------------------|----------|
| N                                |  100000  |
| h                                |    9     |
| b                                |    9     |
| H                                |   512    |
| B                                |   512    |
| Probability of Routing in 2 hops |   100%   |
| Routing State Size               |   390    |

> We've omitted other sizes such as 1M to 1B just due to the mere fact that Excel can't support that much and we will need to use a more sophisticated tool.

##### Optimal theoretical parameters for a 5M peers network:

| Parameter                        |           |
|----------------------------------|-----------|
| N                                |  5000000  |
| h                                |    10     |
| b                                |    10     |
| H                                |   1024    |
| B                                |   1024    |
| Probability of Routing in 2 hops |   99%     |
| Routing State Size               |   9765    |

##### Real life experimentation data

| Parameter                        |           |
|----------------------------------|-----------|
| N                                |  6000     |
| h                                |    5      |
| b                                |    5      |
| H                                |   32      |
| B                                |   32      |
| Probability of Routing in 2 hops |   94%     |
| Routing State Size               |   375     |

Commentary: 

In a simulation involving 2000 randomly picked peers out of 6000:

 * 1701 Requests for unique routing performed (_no redundant peers_)
 * 1641 routes happened in 2 hops
 * 60 routes happened in 1 hops

Meaning: 94% of the request happened in 2 hops, whilst the rest happened in exactly 1 hop, with 0 undefined behavior cases.

(_The real life simulation is available at [hydrate/examples/routing-simulation.go](https://github.com/pokt-network/hydrate/blob/main/examples/cmd/gemini/routing.go)_)

### 4. Gemini Limitations
----

#### 4.1 An Elastic Network

##### Description
As network size is not a constant, but rather grows and shrinks, we are interested in accounting for the edge case when there aren't as many peers in the network, which leads to some peers being the only peers in a their respective hat or boot clubs, making them lonely unreachable islands in terms of routing.

To further gain insight on such behavior, we've ran a little experiment to see at which node count do lonely islands appear and disappear. (Code available at [Hydrate/examples/distribution.go](https://github.com/pokt-network/hydrate/blob/main/examples/cmd/gemini/distribution.go))

The results were as follows:

| Node Count | FIsolated | PIsolated | RF %  |
| :--------- | :-------: | --------: | ----: |
| 50         | 2         | 8         | 0.92  |
| 100        | 0         | 4         | 0.66  |
| 150        | 0         | 1         | 0.57  |
| 200        | 0         | 0         | 0.22  |
| 250        | 0         | 0         | 0.26  |
| 300        | 0         | 0         | 0.02  |
| 350        | 0         | 0         | 0.01  |
| 400        | 0         | 0         | 0     |
| 450        | 0         | 0         | 0     |
| 500        | 0         | 0         | 0     |
| 550        | 0         | 0         | 0     |
| 600        | 0         | 0         | 0     |
| 650        | 0         | 0         | 0     |
| 700        | 0         | 0         | 0     |
| 750        | 0         | 0         | 0     |
| 800        | 0         | 0         | 0     |
| 850        | 0         | 0         | 0     |
| 900        | 0         | 0         | 0     |
| 950        | 0         | 0         | 0     |

![Data showing isolation increase/decrease according to network growth](https://i.ibb.co/M6LsynB/Screen-Shot-2021-09-15-at-1-29-53-AM.png)


We can clearly state that routing failures start to disappear at 300 peers, but that does not mean we are at maximal routing efficiency. (_Some routes were happening after 100 hops_)

To be able to accommodate for the phase when the network is either still growing and below 1000 (_we are setting 1000 as an example of when routing is picking up towards maximal efficiency_) or has shrunk down, we would like to implement a few intermediate strategies to ensure that the network (_even at the worst of partitions, say 50 nodes are the only nodes alive_) we can still perform chain capabilities flawlessly.

To further give an idea about how the routing efficiency behaves in function of the network size (_for h=b=5_), we present the following graph:

![Hr in Function of Network Size](https://i.ibb.co/dm3RHys/Screen-Shot-2021-09-15-at-1-58-34-AM.png)

For this, we would like to fallback to using a mechanism or logic of routing state maintenance such that the network state in all nodes is covering a maximal number of nodes.

We can think of this as: the smaller the network, the broader the scope of the peer (_the larger the routing state_).

##### Formalization

To restate the problem in clearer terms, we can say the following:
* **Lemma**: Given Gemini default parameters, the smaller the network, the more hat clubs that are not covered by any other boot clubs (_pointer groups basically_), hence the isolation.

To further contextualize this limitation, we would like to describe the growth of the network in terms of three phases:

1. Expansion phase (_High entropy, low accuracy_)

This phase is characterized primarily by the network peer count being less than **4000** peers.
This phase is the most problematic as far as Gemini is concerned and represents the phase where the limitation at hand is susceptible to occur.

2. Rapid growth phase (_Entropy near median, accuracy near optimal_)

This phase is characterized primarily by the network peer count being between **4000** and **100K** peers.
Gemini at this stage is approximation high accuracy at a rapid pace, it is safe to say per our data that past **6000** peers, setting h=b=5 will ensure at least 92% routing probability. Past *10K* will perhaps require an increase for `h` and `b` for optimal maintenance cost and hyper-accuracy rates.

3. Stabilization phase (_Low entropy, Maximal accuracy_)

This phase is characterized primarily by the network peer count being between **100K** and **5M** peers. This should represent the plateau phase for Gemini, in this phase, we will defacto fallback to using the recommended paper parameters. No particular hinderances should occur at this phase.

### 5. Solution: Double Affinity & Pointers Group
---
#### 5.1 Affinity & Pointer Groups
If we closely look at Kelips (_and OneHop/UnoHop and other constant hop algorithms_) and Gemini, they are both trying to solve the same problem with almost the same approach, except that Gemini is more ingenious and dynamic due to the mere fact that it relies on uniform distribution laws.

The main similarity is that the entire address space is categorized into multiple "affinity groups" and access to other affinity groups from a given particular group is guaranteed thanks to pointer groups.

Kelips explicitly defines and maintains the pointers groups while Gemini employs items from the second dimension, to the other affinity groups (_hats_), which requires no explicit maintenance of definition.

This abstraction is super useful and reduces the problem to:

If my node is in Affinity Group X:

 - What is the probability of finding a random node Y in this affinity group?

 - How many other affinity groups is this group connected to and what is the probability of finding node Y in those other groups?

Gemini just outstandingly performs well when it comes to solving this.

So when we are dealing with a disperse network with fewer peers that risk being entirely isolated and not connected either from the first dimension (_their own affinity group_) or from the second dimension (_by being pointed at from a pointer group_), all we need to do is increase the probability of finding our isolated nodes through some mechanism, thus breaking that isolation although not explicitly.

##### 5.2.1 Exploiting multi-dimensionality

To accommodate for a very dispersed network or a small network, technically a network at the **expansion phase**, we simply suggest to increase the dimensions through which we establish perspectives about the network.

For every node that has an affinity group (_hat club_) and a pointers group (_boot club_) to other affinity groups, we will issue a second affinity group and a second pointers group such that, the second affinity group is a third dimension group allowing for a completely and entirely new view of the network and a unique membership set(_meaning more probabilities_) and a pointers group as a fourth dimension group that simply points to the other affinity groups at the third level.

In short, this will just double the probabilities and chances of a network to route in a given request in 2 hops due to the mere fact that a node will magically now belong to two unique affinity groups, both of which have second-dimension (_relative to their dimension_) employed items as pointers.

To put this in concrete terms, take a peer A from the network, peer A has:

##### 1. a hat club (_first dimension affinity group_):

This relationship is possible thanks to the fact that peer A has a prefix of a length `h`, let's assume it's `00100` for the sake of the example.

##### 2. a boot club (_second dimension items group to point to first dimension affinity groups_):

This relationship is possible thanks to the fact that peer B has a suffix of a lenght `b`, let's assume that it is `00000` for the sake of the example, all addresses that belong to an affinity group are pointed at naturally by the suffix they have, hence the possibility of pointers employing.

##### 3. a shadow hat club (_a third dimension affinity group_):

This relationship is possible by one magical trick we play, that is to "reverse" prefixes and suffixes to achieve this. Meaning, the suffix of this peer, will now act as a prefix, and thus provide this peer a new hat club, however throuh an inverted relationship so to say.

##### 4. a shadow boot club (_a fourth dimension items to point the third affinity groups_):

This relationship is possible by the same trick as mentioned in the shadow hat club, only as you can already tell, this time around, the suffix will act as a prefix. Meaning that the third dimension affinity groups are going to be pointed at by this fourth dimension group items.

Running a [rounting simulation](#) for 100 peers in a network with h=b=3 resulted in (_100 runs_)


| Node Count            | 100   |
| :-------------------- | :---: |
| h                     | 3     |
| b                     | 3     |
| Routed in 2 hops (%)  | 66%   |
| Routed in 1 hop (%)   | 28%   |
| Routed in > 2 hops(%) | 3%    |


##### 5.2.3 Formalization

To formalize this solution, it would be useful to first vulgarize how will the algorithm, the discovery process and the churn management process change then describe them in a formal way.

##### 5.2.3.a Routing Algorithm

In the default settings Gemini, the routing task can be reduced to the simple statement:

> If the destination peer is in the same affinity group as the routing peer, then done, otherwise, route me to another affinity group.

The rest is just probability, and when fine-tuned and well-parameterized, we approximate a 99% chance of finding our destination node in just two hops, meaning the immediate next affinity group. So many actually get responded to within 1 hop.

The fourth dimension approach is no different, except that the previous statement will gain a second dimension to it, so it becomes:

> If the destination peer is in the same first-order affinity group as the routing peer, then done, otherwise, check if it is in the second-order affinity group, if yes then done, otherwise, route to another affinity group that is in the same dimension as the first order one, if none, try with other affinity groups that are in the same dimension as the second order one.

The solution in short is just an extension to the collision space possible between a given address picked from a set E and an already defined sub address space partial of the set E with a 99% probability of collision/intersection.

The algorithm then becomes as follows:

![Four-dimensional Gemini](https://i.ibb.co/J5yg0xm/Screen-Shot-2021-09-21-at-4-41-48-AM.png)


##### 5.2.3.b Discovery

The Discovery Process's main goal is to first:

1. Decide on the Gemini dimension in which the network will operate

   Meaning, the seed node will determine whether peers will have a routing table consisting of just two symmetrical parts one representing the first-order affinity group (_aka hat club_) and the second-dimension pointers to the first-order affinity-groups (_aka boot clubs_) or four, the two just aforementioned + a second-order affinity group and its second dimension pointers group.

2. Provide joining peers with members or members lists representing each Gemini dimension respective to the peer.

  Meaning, the seed node will exchange with the joining peer at least one member for each dimension of his routing table, for the joining peer to ask that member for the members list respective to their group.

For the seed node to determine the level, we will simply set the following condition:

A. If the network is still in an **Expansion phase**
   1. Gemini dimension will be 4 instead of default 2.
   2. h and b will `h=b=3`
B. If the network is at a **Rapid Growth phase**
   1. Gemini dimension will be back to default, 2.
   2. h and b will then use the correct estimated values per our data sheets, `h=b=5`, we might opt for `h=5 b=3` for more accuracy.
C. If the network is at a **Stabilization phase**
  1. Gemini dimension will be the default, 2.
  2. h and b will use the paper parameters, `h=b=10`

More on how the seed node is able to determine such information and trigger a change event in [Peer Logic and Features / Network Roles / Seed Node](https://github.com/pokt-network/hydrate/wiki/Peer-Logic-And-Features#1-seed-nodes)

##### 5.2.3.c Churn Management

As stated in the [Churn Management](https://github.com/pokt-network/hydrate/wiki/Churn-Management) chapter, the default Gemini behavior is to reduce the management hassle to the specific scope of the node's affinity group and pointers group, aka hat and boot club.

Nothing much will change except that in expansion phase of the network, a peer will maintain two extra clubs, which will double the maintenance cost in terms of bandwidth and messages received per second by the node, but it won't be much regardless.

If you've visited the elaborations we've done in the churn management chapter, the new values will be just double the previous ones, meaning:

* 12 messages per second per node
* 6kbps will be consumed by the maintenance process

## IV. How Does Gemini influence the other parts
----

1. Club (_hat and boot_) elements are interconnected, meaning that there is a cost to maintaining that interconnected, how does this influence outbound/inbound connections count and pooling?
   A. Answer available at [Transport Logic And Security](https://github.com/pokt-network/gemelos/wiki/Transport-Logic-And-Security) chapter.