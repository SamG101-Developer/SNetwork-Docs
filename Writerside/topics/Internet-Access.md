# Internet Access

The route formed using nodes in the DHT is used to forward data leaving the node, on specified ports. For the Internet
(HTTPS), this would be port 443. The data is captured using the Socks5 protocol, encrypted in layers, and sent to the
entry node of the route. Other protocols, such as FTP, can be registered for interception too.

Each node removes a layer of encryption, and forwards the data to the next node in the route. The exit node sends the
data to the destination server, and the response is sent back to the client in the reverse order, each node adding a
layer of encryption. The client will remove the layers of encryption, and parse the response.

