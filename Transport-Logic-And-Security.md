## Overview
----

Transport logic and security are key elements on the inner working of the p2p network. Here we try to outline the general properties and specifications that our network should have and comply with. We also detail some possible attacks that we may be susceptible to. 


## Specification
----

### 1. Connection Lifecycle

* A connection is initiated by the peers
* Handshake protocol is initiated and peers exchange secrets to establish a secure encrypted channel.
* Msgs are then sent on-demand while the connection is alive.
* The connection uses a default timeout to ensure that if idle for x amount of time resources are freed and no unnecessary allocations happen.

**handshake protocol draft:**

1. peers send each other their ephemeral Ed25519 public keys,
2. peers convert the public keys they received into Curve25519 public keys,
3. peers convert their ephemeral Ed25519 private keys into Curve25519 private keys,
4. peers establish a shared secret by performing ECDH with their private Curve25519 private key and their peers Curve25519 public key,
5. peers use the shared secret as a symmetric key and communicate from then on with messages encrypted/decrypted via. AES 256-bit GCM with a randomly generated 12-byte nonce.


### 2. Connections Pooling

* basic bounded connection pool for regular operation with persistent peering options.

* Specialized bounded pools for application use cases:
   * syncing allowance
   * consensus tasks (validators)
   * Others.

### 3. Protocol

TCP/IP with a handshake for direct communications
UDP for gossip-like / group or network-wide communication

### 4. Multiplexing
Multiplexing can be used by an implementation to take advantage of already existing connections and send different messages through the same channel. 

## Security
----
Peer connections could be encrypted using AES 256-bit Galois Counter Mode (GCM) with a Curve25519 shared key established by an Elliptic-Curve Diffie-Hellman Handshake.

Very similar to TLS handshakes

### 1. Attacks

**Eclipse attacks**

Eclipse attacks are a special type of cyberattack where an attacker creates an artificial environment around one node, or user, which allows the attacker to manipulate the affected node into wrongful action. By isolating a target node from its legitimate neighboring nodes, eclipse attacks can produce illegitimate transaction confirmations, among other effects on the network. While these types of attacks isolate individual nodes, the effectiveness of eclipse attacks at disrupting network nodes and traffic largely depends on the structure of the underlying network itself. Importantly, eclipse attacks are extremely rare in the real world; the structure of a decentralized blockchain itself generally precludes them.

_<< Selected Overlay give some protection against this types of attacks by using Deterministic node selection but if implementation need more flexibility we can use Random node selection and increased node connections to mitigate the possibility of connecting to an attacker-controlled node >>_

**Routing attacks**

Since each node plays a role in routing traffic through the network, malicious users can perform a variety of "routing attacks", or denial of service attacks. Examples of common routing attacks include "incorrect lookup routing" whereby malicious nodes deliberately forward requests incorrectly or return false results, "incorrect routing updates" where malicious nodes corrupt the routing tables of neighboring nodes by sending them false information, and "incorrect routing network partition" where when new nodes are joining they bootstrap via a malicious node, which places the new node in a partition of the network that is populated by other malicious nodes.

_<< This attack requires the use of the routing capabilities of the overlay to work or try to exploit the DHT, at this moment we are not intending to use routing to send information to other nodes and we are opting for a direct communication approach, these attacks can be also mitigated by using an additional source of truth >>_