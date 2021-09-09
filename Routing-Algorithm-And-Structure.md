## Overview
----
Choosing the proper data structure to represent the structure of a network's overlay is the main and the crucial step to achieving a structured overlay, and a detrimental one for building an efficient and performant network.

In this section, we go try to explain why we chose Gemini as the main structure and algorithm as well as describe a spec for implementation that we hope is going to serve as a reference.

Before we dive in this section, we want to allude to the fact that the paper itself lacks any reference implementation, and is not very rigorous in defining Mathematical terms (_We've had access to three different versions_), so we will provide our own answers in multiple cases, so we want to make use of the following visual toolbox (_while thanking devp2p spec for the inspiration_):

✍🏻: A new term we chose to use for convenience purposes

🗝: Key concept that should be paid attention to


## What is Gemini?
----

Gemini is a structure with an algorithm aimed to support efficient and scalable structured overlays.
Its name comes from the fact that its routing table consists of two parts, one containing nodes with common prefix and the other containing nodes with common suffix. Gemini routes messages to their destinations in just 2 hops, with a very high probability.
Although Gemini is not formally a O(1) routing algorithm, however it can be categorized as so. The same can be said about it if we want to categorize it as a logarithmic degree mesh or a PRR algorithm. 
In our opinion, it has gotten the best of all worlds.

You can find more information about Gemini in the following papers:

  1. [Gemini Paper first version](https://link.springer.com/content/pdf/10.1007%2F978-3-540-24685-5_22.pdf)
  2. [Gemini Paper second version](https://www.researchgate.net/publication/221601988_Gemini_Probabilistic_Routing_Algorithm_in_Structured_P2P_Overlay/link/53fdb64c0cf22f21c2f82a31/download)


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
* **Root node / Destination node**: The destination ID of a message M is the _**numerically closest**_ ID to M's target.
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
    1.b If not, N looks for a node E in its boot club such that E's ID is in the same club as in M's destination ID.
      1.b.a If found, node N routes the message M to node E
      1.b.b If not, node N routes E to a randomly picked node from its hat case such that E's ID is in a different boot case than M's destination ID.

Written in pseudo-code, the Gemini routing algorithm is as follows:
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
 
  E <- null
  for every e in HatClub(N):
    if not (E belongs to BootClub(destID))
    then:
      E <- e

  Route to E.
```

### 4. Maintenance
----

## How Does Gemini Impact The Other Parts
----

1. Club (_hat and boot_) elements are interconnected, how does this influence outbound/inbound connections?

### Open questions

1. What happens if I am the only element in my hat and boot club?
   * lowest h and b
   * multi level routing table (h=b=5, h=b=4, h=b=3, h=b=2)
