# Bootstrapping

When a node joins the network for the first time, they will generate an asymmetric key pair. The hash of the public key
is used as the node ID. The node will connect to one of the directory nodes; these are hardcoded IP addresses that are
known to be online and are part of the network. Their public keys are also hardcoded in the client. The node will create
a X.509 certificate with the public key and send it to the directory node.

The directory node will then sign the certificate with its private key and send it back to the node. The node will then
use the certificate to connect to the network. The directory node will also send the node a list of other nodes that are
online. The node will then connect to these nodes and exchange certificates with them. The node will then be part of the
network and will be able to communicate with other nodes.

If someone intercepts the node's X.509 certificate request to the server, and changes the public key in the certificate,
then its hash won't match the ID on the certificate. The directory node will know that the certificate is invalid and
will not sign it. The node will not be able to connect to the network.

If the public key and the ID on the X.509 certificate are modified, such that the ID matches the hash of the modified
public key, then the certificate will be signed by the directory node, but when the original node receives the signed
certificate, it will check the ID on the certificate against the locally stored ID. As they don't match, the certificate
will be discarded, and the node will send a new certificate request to the directory node.

Tampered certificates (certificates of tampered keys and IDs) don't clog up or affect the DHT in any way, because the
directory service doesn't add nodes to the DHT; the nodes themselves, having been authenticated, add themselves to the
DHT.

## Protocol

1. Node Connection Preparation
    - Generate an asymmetric key pair: `(ePK, eSK)`
    - Create a X.509 certificate `Cert` with `(ID, ePK)`
    - Create a JSON request `R` with `(CertificateRequest, Cert)`
    - Send `R` to the directory node

2. Directory Node Handles the connection request `(CertificateRequest, Cert)`
    - Ensure the hash of `ePK` matches `ID`
    - Sign the certificate with the private key of the directory node to create `S0`
    - Create a JSON response `R` with `(CertificateResponse, S0, NodeList)`
    - Send `R` to the node

3. Node Handles the connection response `(CertificateResponse, S0, NodeList)`
    - Verify `S0` with the public key of the directory node
    - Save the certificate `Cert`
    - Save the list of nodes `NodeList`
    - Connect to the nodes in `NodeList` and exchange certificates

4. Node is now part of the network
    - The node can now communicate with other nodes in the network
