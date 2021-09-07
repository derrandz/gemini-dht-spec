## Overview

The dynamics of peer participation, or churn, are an inherent property of Peer-to-Peer (P2P) systems and critical for design and evaluation. Accurately characterizing churn requires precise and unbiased information about the arrival and departure of peers, which is challenging to acquire due to the nature of the P2P systems.
----

## What is Churn Management?

We can briefly define churn as the process of peers leaving and entering the system continuously. Churn Management tackles the common tasks based on the interactions of the peers.
----

## Specification
----

#### 1. Discovery

Peers need to be able to discover other Peers based on their perspective of the network.

Our Overlay should allow that any Peer A is able to reach any other Peer B even if Peer B is not on a direct communication path with Peer A.

For the current overlay, Gemini, the routing table is divided into 2 parts, All Peers on each are considered to be on a direct communication path between them. All other nodes that are reachable through hops are said to be on an indirect communication path.

This allows for an on-demand discovery of peers through hops, harnessing the ability of the overlay to reach any node on the network on 2 hops in an almost constant manner (probabilistic routing). Is important to distinguish that even if the Peer is discovered by using hops on the P2P network (indirect communication path), after the discovery a direct communication is possible, details about how to archive this depend on the implementation and the inherent peer maintenance costs associated.

A Specialized Peer type that helps the standard Peer on discovery tasks should exist if we need advanced capabilities for discovery. But a basic defined logic should exist on all peers that allow the network to not depend on the specialized peer for basic performance.

#### 2. Join

When a new Peer X joins the network, it first contacts an existing bootstrap peer E. Peer E picks a Peer H with X’s hat(prefix) from its bootcase(suffix group), as well as a Peer B with X’s boot(suffix) from its hatcase(prefix group). X collects its hatcase from H and bootcase from B. If E does not find appropriate H or B, it asks another Peer in its hat or bootcase for help.

This way a Peer that joins is paired with its corresponding Prefix and Suffix groups, which should provide sufficient peering for the application needs.

#### 3. Leave

A peer that wants to leave the network basically disconnects relies on the maintenance routine to broadcast its unavailability. Every x unit of time defined by the maintenance params, the Peers on each individual group (suffix or prefix) ordered in a Ring-like or linked list structure verify the status of the next peer in the list (Last Peer verify the first) by sending a "heartbeat" message. If T failures happened the Peer then multicasts the unavailability of the verified Peer to all the nodes in this group.

#### 4. Maintenance



