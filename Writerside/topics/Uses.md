# Uses

Every node's certificate is stored on the network, with their node identifier has the key. This allows for the DHT to
double up as the Public Key Infrastructure (PKI) for the network. This allows nodes to get the certificates of other
nodes without having to connect to that node, or a centralized server, to get the certificate. This is useful for
authenticating tunnelled connections, where the client is not directly connected to the target node, and needs to remain
anonymous from the target node.

The DHT also stores files, in 2 different ways:

- [Hidden Services](Hidden-Services.md)
- [Distributed Files](File-Storage.md)

Every resource accessible over the DHT has a duplication flag. If this flag is set, then the resource is stored on
multiple nodes, to ensure availability. If the flag is not set, the resource is stored on a single node. Typically,
files like images and documents would be stored with the duplication flag set, while files like websites would be stored
without the duplication flag set.
