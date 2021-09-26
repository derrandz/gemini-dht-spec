# The Bootstrap Node

A special bootstrap node should exist in the network to help with tasks that require more precise knowledge of the networks as a whole.

The tasks that this specialized peer will be trusted with are the following:

General Network
1. Explore the network and gain general information from peers.
    * the bootstrap node will perpetually try to connect to every peer on the network periodically, to gather peer information about the peers and their groups.
2. Keep a regularly updated chart of the network. ( group number, group size )
    * the bootstrap node using the information gathered from exploring the network should be able to create a map or chart of the network that can work as a Master Routing table as it will contain the information from all the different groups
3. Help peers locate their affinity groups.
    * the bootstrap node will be part of the de facto known nodes and will allow new peers to locate their affinity groups.  
4. Help peers locate other peers in different groups
    * If by using the default routing/discovery algorithm a peer is not able to locate or send a message to a peer on other group seed nodes should be able to provide the requested peer's last known information.

Chain Aware Network    
5. Respond to Chain-awareness Sync Request from peers
    * Although all the nodes are part of chain-aware network The bootstrap node network chart is expected to be updated at regular intervals and can help with cross-group syncing scenarios.


