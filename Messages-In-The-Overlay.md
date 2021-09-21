### Overview
---
The network should support multiple message types to fit the different use cases.
Some of them are direct communication, intelligent gossip like messaging, and low bandwidth maintenance messages. A top layer of Event-Based Communication could be implemented by the application and paired with these message types.

### What is Messages and Message Types
---

### Why Messages and Message Types
---

### Specification
---

#### Types:

1. Peer Message
2. Direct Message
3. Passaround Message
4. Targeted-Passaround
5. Chain-awareness Sync Request
6. Network Event Message

#### 1. Direct Message
A general-purpose message with a specific destination. Basic private data exchange between peers. Usually delivered by Establishing a connection between peers.

#### 2. Passaround Message

Messages that are deemed of interest to the network match these types of messages. The trickle around broadcasting guarantees that all the groups are covered by delivering the copy of the message to each one of them and then individually propagating the message within the groups.

#### 3. Network Event Message

The message used to inform other peers of a member-changing event is called a Peer event message.

This message is used to share the joining of a new peer in a group and by maintenance to report unavailability or change of peer configuration.

This message is usually bound to a group and broadcasted to the members.
