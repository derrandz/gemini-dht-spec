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
----

In this section, we will start by defining the main terms used by the paper to remove any ambiguity. Those will be used heavily in this specification and we hope in any implementing code.

* **Hat case**: first `h` bits of a node's ID
* **Boot case**: last `b` bits of a node's ID
* **Node id**: an ID resulting from _uniformly distributed hash function*_ 
* **Numerically close**: A node A is closest to node B if: _for every node x out of node A's neighbors `|IDa - IDb| < |IDa - IDx|`_
* **Numerically closest** (🗝): node A's ID is the numerically closest node to itself such that: `|IDa - IDa| = 0 and 0 < |IDa - IDx|`
* **Hat club**: The set of nodes such that their IDs first `h` bits are the same as concerned node
* **Boot club**: The set of nodes such that their IDs first `b` bits are the same as concerned node
* **Root node / Destination node**: The destination ID from a message M such that M's destination ID is _**numerically closest**_ to the concerned node's ID

### 2. Routing Data Structure
----

### 3. Routing Algorithm
---

## How Does Gemini Impact The Other Parts
----

