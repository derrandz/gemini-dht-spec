Welcome to the gemelos wiki!

Gemelos is the p2p layer powering Pocket Network 1.0

In this Wiki all information relative to the Gemelos specification and implementation, respecting the following order:

1. [Routing Structure and Algorithms](https://github.com/pokt-network/gemelos/wiki/Routing-Structure-And-Algorithms)
2. [Churn Management](https://github.com/pokt-network/gemelos/wiki/Churn-Management)
3. [Node Identification & Security](https://github.com/pokt-network/gemelos/wiki/Node-Identification-And-Security)
4. [Transport Protocols & Security](https://github.com/pokt-network/gemelos/wiki/Transport-Protocols-And-Security)
5. [Peer Logic & Features](https://github.com/pokt-network/gemelos/wiki/Peer-Logic-And-Features)
6. [Usability Interface / Interfacing with Other Modules](https://github.com/pokt-network/gemelos/wiki/Usability-Interface)

To make sense of the aforementioned segmentation of specifications, we present this laundry list of features a p2p library might have and how they can get segmented as was done above:

1. find peers [Churn Management]
2. classify peers (nodes, validators, full nodes, Listeners) **[Peer Logic & Features]**
3. Help other nodes find peers (answers peering info requests) **[Churn Management]**
4. Measure peer response times and lack of response **[Churn Management]**
5. Receive messages and route them to correct module or message queue. **[Routing Structure and Algorithm(s)]**
6. Enforce peer bans **[Peer Logic & Features]**
7. Send messages according to message type (direct, gossip, multicast) **[Peer Logic & Features]**
8. Collect peer height and sync state. **[Peer Logic & Features]**
9. Handle and dispatch block sync requests from other modules. **[Peer Logic & Features]**
10. Handle and reply to block requests from peers. **[Peer Logic & Features]**

And so on and so forth. By no means this is an exhaustive list, but rather an example to help remove jargon fever if the segmentation in list feels too abstract.