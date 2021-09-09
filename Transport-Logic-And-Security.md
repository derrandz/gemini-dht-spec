## Overview
----


## Transport Logic
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

//WIP Review. --- taken from Perlin/Noise


### 2. Connections Pooling
//wip detail inner working

* basic bounded connection pool for regular operation with persistent peering options

* Specialized bounded pools for application use cases:


### 3. Protocol

TCP/IP with a handshake for direct communications
UDP for gossip-like / group or network-wide communication

### 3. Multiplexing
TBD

## Security
----
Peer connections could be encrypted using AES 256-bit Galois Counter Mode (GCM) with a Curve25519 shared key established by an Elliptic-Curve Diffie-Hellman Handshake.

Very similar to how TLS handshakes