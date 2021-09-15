# This is a living page, we've very recently identified a serious optimization so this is for now not ready for consumption.

## Overview
----
Choosing the proper data structure to represent the structure of a network's overlay is the main and the crucial step to achieving a structured overlay, and a detrimental one for building an efficient and performant network.

In this section, we go try to explain why we chose Gemini as the main structure and algorithm as well as describe a spec for implementation that we hope is going to serve as a reference.

Before we dive in this section, we want to allude to the fact that the paper itself lacks any reference implementation, and is not very rigorous in defining Mathematical terms (_We've had access to three different versions_), so we will provide our own answers in multiple cases.

**A reminder of our visual toolbox** (_while thanking devp2p spec for the inspiration_):


💡: indicates an implementation avenue or suggestion

✍🏻: A new term we chose to use for convenience purposes

🗝: Key concept that should be paid attention to

## (WIP) Problematic
----

## (WIP) Proposed Solution: Constant Hop Affinity DHTs (CHAD)
----

In short: CHAD = Kelips + Gemini

## (WIP) Why CHAD?
----

### (WIP) What is an Affinity DHT?

#### (WIP) Kelips

#### (WIP) Gemini

#### (WIP) CHAD

## What is Gemini?
----

Gemini is a structure with an algorithm aimed to support efficient and scalable structured overlays.
Its name comes from the fact that its routing table consists of two parts, one containing nodes with common prefix and the other containing nodes with common suffix. Gemini routes messages to their destinations in just 2 hops, with a very high probability.
Although Gemini is not formally a O(1) routing algorithm, however it can be categorized as so. The same can be said about it if we want to categorize it as a logarithmic degree mesh or a PRR algorithm. 
In our opinion, it has gotten the best of all worlds.

You can find more information about Gemini in the following papers:

  1. [Gemini Paper first version](https://link.springer.com/content/pdf/10.1007%2F978-3-540-24685-5_22.pdf)
  2. [Gemini Paper second version](https://www.researchgate.net/publication/221601988_Gemini_Probabilistic_Routing_Algorithm_in_Structured_P2P_Overlay/link/53fdb64c0cf22f21c2f82a31/download)
  3. [Kelips Paper](http://iptps03.cs.berkeley.edu/final-papers/kelips.pdf)

## Why Gemini?
----
 
1) Gemini divides peers into a symmetric manner, allowing for maximum control and configuration.
2) Gemini nodes employ items in the other dimension of groups as pointers allowing peers to have a reduced fixed scope with high "dispersed-ness", which means most of the network will be covered in that small scope. (smaller routing table).
3) In Gemini whether a message can successfully reach its target in one hop is in some probability. Reducing this probability can trade hop count for bandwidth overhead, making Gemini more scalable than others.
4) Unlike other overlays that are designed for file lookups, Gemini is just a substrate, making it lighter and leaving data operations to its application designers, and making it an ideal candidate for a role such as "Network backbone".
5) Gemini is simple, no specifically complex classification or routing logic is required, Gemini relies on natural uniformity and distribution laws that are observably always true as long as parameters are respected, which significantly reduces human-error chances.
6) No better candidate! You can refer to the our research summary to learn more about this. // TODO: Link to Forum summarizing our research effort or add in as a separate page in the wiki

## Specification
----

### 1. Glossary

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

### 2. Routing Data Structure

The routing data structure is comprised of a hat club and a boot club representation of the network relative to the concerned node, and a formal way of describing it would be:

```
Rt(A) = { HatClub(A) U BootClub(A) }
```

Meaning, the routing table of a node A is the union of its hat club and boot club.

Let's start by describing the hat and boot clubs and how they make up a routing table.

#### Hat Club

We define `n` as the length of IDs assigned to node, A hat club is represented as an address space (N>>2^n) such that each ID belonging to a hat club shares the same `h` bits with the concerned node's ID.

In a given hat club:

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to the head's ID (representing "the start point on the ring").
  * IDs partaking in a hat club are all unique.
  * A hat club can measure the distance between a given ID and its head's ID.
  * A hat club can calculate the **numerically closest* ID(s) to its head.

#### Boot Club

We define `n` as the length of IDs assigned to node, A boot club is represented as an address space (N>>2^n) such that each ID belonging to a boot club shares the same `b` bits with the concerned node's ID.

In a given boot club:

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to center's ID (representing "the start point on the ring").
  * IDs partaking in a boot club are all unique.
  * A boot club can measure the distance between a given ID and its head's ID.
  * A boot club can calculate the **numerically closest* ID to its head.

### 3. Routing Algorithm
---

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

### 4. Maintenance
----

A peer cares to maintain only its reduced fixed scope of the network, that is its hat and boot clubs. To be able to maintain an updated view of the network, a Gemini peer sends periodic heartbeats to its clubs members and updates it state accordingly.

To see the full details of this process, please refer to the [Maintenance](https://github.com/pokt-network/hydrate/wiki/Churn-Management#4-maintenance) Section in [Churn Management](https://github.com/pokt-network/hydrate/wiki/Churn-Management) Chapter of this wiki/spec.

### 5. Network Parameters and Scalability
----

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

### 7. Edge cases
---

#### Elastic Network

##### Description
As network size is not constant, but rather grows and shrinks, we are interested in accounting for the edge case when not as many peers in the network lead to a peers being the only peers in a their respective hat or boot clubs, making them lonely unreachable islands in terms of routing.

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

To be able to accommodate for the phase when the network is either still growing and below 1000 (_we are setting 1000 as to when routing is near maximal efficiency_) or has shrunk down, we would like to implement a few intermediate strategies to ensure that the network (_even at the worst of partitions, say 10 nodes are the only nodes alive_) we can still perform chain capabilities flawlessly.

To further give an idea about how the routing efficiency behaves in function of the network size (_for h=b=5_), we present the following graph:

![Hr in Function of Network Size](https://i.ibb.co/dm3RHys/Screen-Shot-2021-09-15-at-1-58-34-AM.png)

For this, we would like to fallback to using a Kelips' O(1) logic of maintaining the whole network state in all nodes for small network sizes.

We can think of this as: the smaller the network, the broader the scope of the peer (_the larger the routing state_).

##### Formalization

// Todo discuss with other whether such state is readily available or rather on-demand from a seed node.


## How Does Gemini Impact The Other Parts
----

1. Club (_hat and boot_) elements are interconnected, meaning that there is a cost to maintaining that interconnected, how does this influence outbound/inbound connections count and pooling?
   A. Answer available at [Transport Logic And Security](https://github.com/pokt-network/gemelos/wiki/Transport-Logic-And-Security) chapter.