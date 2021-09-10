## Overview
----

The dynamics of peer participation, or churn, are an inherent property of Peer-to-Peer (P2P) systems and critical for design and evaluation.  The maintenance of an up to date view of the network from a peer's perspective is an elaborate process in itself and is a must to be able to accommodate for the aforementioned dynamics and keep the network alive. A process that every participant must perform to the best of its effort.

We call this process, Churn Management.

## What is Churn Management?
----

We can briefly define churn as the process of peers leaving and entering the system continuously. Considering that churn is primarily a change in the state of the network, churn management is a state management process that aims to keep the network state up to date. Provided that the network state is the sum of all the partial networking states of its participants, churn management is then the process of keeping an up to date networking state at the participant level.

## Why Churn Management?
----

As per the previous section, without a churn management process, there will effectively be no network. 

Since it is not a question of choice whether to implement a churn management process, and since it is an indispensable part of any peer-to-peer network, in our specification, our efforts aim to rather define an "efficient" and "cheap" churn management process.

## Specification
----

### 1. Discovery

#### A. Breakdown

Peers need to be able to discover other peers based on their perspective of the network.

Our overlay should allow that any peer A is able to reach any other peer B even if peer B is not on a direct communication path with peer A.

Given the current structure of the overlay, Gemini that is, the routing table is divided into two parts. Every peer on each part is considered to be on a direct communication path with every other peer in that part, _i.e: all peers in a given part of the two parts of the routing table are fully interconnected_ .

All the other peers that are reachable through more than one hop and are not directly reachable from the state of the concerned peer, are said to be on an indirect communication path with this peer.

This allows for an on-demand discovery of peers through hops, harnessing the ability of the overlay to reach any node on the network only in two hops in an almost constant manner (_probabilistic routing_).

It is important to distinguish that even if the peer is discovered by using more than one hop in the peer-to-peer network (_indirect communication path_), A direct communication is possible after discovery, details about how to archive this depend on the implementation and the envisaged peer maintenance costs.

A specialized peer type that helps the standard peer with discovery tasks should exist if we require advanced capabilities for discovery. However a basic defined procedure be accessible for all peers to allow the network to not depend solely on the specialized peers for basic performance and increased robust-ness and fault-tolerance.

#### B. Formalization

**Requirements**:

1. Any given random peer should be able to discover other peers in the network from their given current perspective of the network.
2. Any given peer should be able to discover/reach indirect peers provided that it has a few direct peers.
3. A specialized peer should be present in the network to further enhance discovery.
4. Any given node can perform basic discovery and can safely fallback to such procedure in the absence of specialized peers in the network.

### 2. Join

When a new peer X joins the network:

1. It first contacts an existing bootstrap peer E.
2. Peer E picks a peer H with X’s hat from its boot club*, as well as a peer B with X’s boot from its hat club*.
3. Peer X retrieves its hat club member list from H and boot club member list from B.
  3.a If E does not find an appropriate H or B, it asks another peer in its hat or boot club for help (_forwards the request_)

This way, when a peer joins, it is immediately paired with its corresponding hat and boot clubs, which should provide sufficient peering for the application needs.

### 3. Leave

A peer that wants to leave the network basically just disconnects and relies on the maintenance routine to "discover" and broadcast its unavailability.

Every `L` unit of time/duration of time --_defined in the maintenance parameters_, peers in each individual club (_hat and boot_) ordered in a ring-like structure verify the status of the next peer in the ring (_last peer verify the first_) by sending a `Heartbeat` message (_Refer to [Messages In The Overlay](https://github.com/pokt-network/hydrate/wiki/Messages-In-The-Overlay) chapter to learn more_).

If `T` failures happened, the peer then multicasts the unavailability of the verified peers to all the other peers in that club, by sending a `Network Event` message.

* 💡 Peers responses to `Heartbeat` messages can be used by an implementation to profile peers aliveness and establish dynamic ban/timeout procedures for faulty nodes as a response against scenarios that could make network conditions unreliable within a group. 

* 💡 The sending of a `Direct` message is an option if an implementation chooses to allow regular peering with elements outside of the hat/boot clubs. This will marginally increase maintenance/bandwith costs but allows for a more flexible communication path if required.

### 4. Maintenance

As explained before, the Gemini overlay routing table consists of two parts, a hat club and a boot club, the first one containing all peers sharing the same h-bits prefix and the latter containing all peers sharing the same b-bits suffix (_h and b are systematic parameters_).

The values for h and b define the size of these clubs and add to the maintenance cost. (_Refer to [Routing Structure And Algorithm](https://github.com/pokt-network/hydrate/wiki/Routing-Algorithm-And-Structure) to revisit Gemini paramters_) 

Gemini adopts a report-based routing table maintenance algorithm.
When a membership change event occurs, the overlay will multicast this event in a report-based mechanism, which helps the overlay consume low bandwidth to deal with peers' join and leave.

When values of `h` and `b` are the same, our maintenance is defined by the following formula:

`M = (4(N) * f ) / (2^b * L)`

where:

* `N = number of peers in the network`
* `f = redundancy of the multicast algorithm`
* `b = number of bits taken for the common suffix`
* `L = average lifetime of peers in seconds`

**Additional elements added to the basic maintenance routine will add to this cost.**
**Also, this cost is per `L` duration of time**

When the maintenance cost is not acceptable by peers, Gemini can also trade hops for bandwidth consumption like other overlays by changing the params to make the routing table smaller to fit the application bandwidth needs.

##### Example scenario:

Assuming that the average lifetime of peers is 1 hour(L), all the items in the routing table have to be refreshed in a period of 1 hour.
It means that for a system using Gemini, which consists of 5,000,000 nodes, and h and b are both set as `10`, on average every prefix/suffix-group contains 5,000,000/2^10 = `4883` peers. 

Then on average, every node receives (4883+4883)*2=`19532` event messages per hour, about `5.43` messages per second. Plus heartbeats and their responses, the total message count per second will not exceed 6 messages per second. 

If Messages size is 500bit, the bandwidth cost average will be around 6 message/second * 500bit = `3kbps`.

