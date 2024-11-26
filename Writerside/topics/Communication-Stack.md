# Communication Stack

The communication stack is a customized network stack used for communicating over the network. It contains specialized
layers inside the typical "application layer". A brief overview of the communication stack is as follows:
- `Application Layer`. This layer is a wrapper for a number of internal application-specific layers, such as a `HTTP`
  layer, or an `FTP` layer. This allows for any application to hook into the communication stack, by providing a common
  interface for applications to use.
- `Routing Layer` This layer is responsible for creating a secure route through nodes in the network. It manages
  end-to-end encryption and authentication, whilst keeping the client anonymous. It is responsible for adding and
  removing layers of encryption.
- `Distributed Layer`: This layer is responsible for communicating with the distributed hash table (DHT). This is
  required to get node addresses or certificates, when building routing tables. This layer has api calls to the DHT for
  storage, retrieval, updating information etc.
- `Connection Layer`: This layer is responsible for establishing and maintaining connections between nodes. It is
  responsible for ensuring that the connection is confidential and authenticated. It is strictly for managing
  point-to-point connections, and extends no further than the two nodes involved.
- `Transport Layer`: This layer is responsible for creating and managing connections between nodes. It is responsible
  for ensuring that the connection is reliable and that packets are delivered in order. It is strictly for managing
  point-to-point connections, and extends no further than the two nodes involved.
- `Interception Layer`: This is a minimal-functionality layer whose responsibility is to intercept packets and pass them
  to the modification layer. No obfuscation happens at this layer, but it is isolated from the modification layer and
  the roles of the two layers are distinct.
- `Modification Layer`: This packet-level layer is responsible for modifying actual packets. Whilst layered encryption
  works at the application level, ie in the routing layer, the packet level encryption works at the modification layer.
  Another responsibility of this layer is to anonymize headers, and inject timing offsets.
- `Internet Layer`: This is the lowest layer in the stack, and is responsible for sending and receiving packets. There
  is no control over this layer, as it is managed by the operating system.
- `Physical Layer`: This is the lowest layer in the stack, and is responsible for sending and receiving packets. There
  is no control over this layer, as it is managed by the operating system.

![](CommunicationStack.svg)