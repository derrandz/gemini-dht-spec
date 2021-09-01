## Overview
----
Choosing the proper data structure to represent the structure of a network's overlay is the main and the crucial step to achieving a structured overlays, and detrimental for building an efficient and performant one.

In this section, we go in depth behind our decision making behind choosing Gemini as the driver behind the structure and the routing algorithm as well as describe a main specification for implementation that we hope is going to serve as a reference implementation.

Before we dive in this section, we want to allude to the fact that the paper itself lacks any reference implementation, and is not very rigorous in defining Mathematical terms, so we will provide our own answers in multiple cases, so we want to make use of the following visual toolbox (_while thanking devp2p spec for the inspiration_):

✍🏻: A new term we chose to use for convenience purposes
🗝: Key concept that should be paid attention to


## What is Gemini?
----
// Link [Paper](https://link.springer.com/content/pdf/10.1007%2F978-3-540-24685-5_22.pdf)


## Why Gemini?
----
// Link to Forum summarizing our research effort or add in as a separate page in the wiki

## Specification
----

### 1. Glossary

In this section, we will start by defining the main terms used by the paper to remove any ambiguity. Those will be used heavily in this specification and we hope in any implementing code.

* **Hat case**: first `h` bits of a node's ID
* **Boot case**: last `b` bits of a node's ID
* **Node id**: an ID resulting from _uniformly distributed hash function*_ 
* **Numerically close**: A node A is closest to node B if: _for every node x out of node A's neighbors `|IDa - IDb| < |IDa - IDx|`_
* **Numerically closest** (🗝): node A's ID is the numerically closest node to itself such that: `|IDa - IDa| = 0 and 0 < |IDa - IDx|`
* **Hat club**: The set of nodes such that their IDs first `h` bits are the same as concerned node
* **Boot club**: The set of nodes such that their IDs first `b` bits are the same as concerned node
* **Root node / Destination node**: The destination ID from a message M such that M's destination ID is _**numerically closest**_ to the concerned node's ID
* **Center of a club** (✍🏻): Is the concerned node in a hat club. If we take node X, node X is the center of its own Hat and Boot Clubs.

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

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to the center's ID (representing "the start point on the ring").
  * IDs partaking in a hat club are all unique.
  * A hat club can measure the distance between a given ID and its center ID's.
  * A hat club can calculate the **numerically closest* ID(s) to its center.

#### Boot Club

We define `n` as the length of IDs assigned to node, A boot club is represented as an address space (N>>2^n) such that each ID belonging to a boot club shares the same `b` bits with the concerned node's ID.

In a given boot club:

  * IDs are ordered on a N>>2^n ring according to their _**numerical closeness**_ to center's ID (representing "the start point on the ring").
  * IDs partaking in a boot club are all unique.
  * A boot club can measure the distance between a given ID and its center ID's.
  * A boot club can calculate the **numerically closest* ID to its center.

### 3. Routing Algorithm
---

To route a given message M, we define the following properties of the message that will be useful to the routing process:

  * A message has a n bits long destination ID
  * A message has a n bits long sender ID

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

