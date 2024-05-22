# Distributed Layer

The Distributed Layer (Layer 3) handles the distributed hash table connections. It manages storing and retrieving files
over the distributed hash table overlay network. As the DHT doubles as the public key infrastructure, it also manages
searching for node keys to authenticate tunnelled connections, where the client is not directly connected to the target
node.

