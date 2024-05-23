# Hidden Services

## Overview

Hidden Services are accessible in a more complex way than [Standard Internet Access](Internet-Access.md). Hidden
Services require a server instance running on a Provider Node. This could be an HTTPS server, an FTP server, or any
other server that can be accessed over the internet. Each resource hosted on the network has an associated ID and public
key (like nodes), which are registered to the network.

## Creating a Hidden Service

When a hidden service is created, the Provider Node must first generate a public key for the service. The public key is
used to create a certificate, which is then signed by the directory node. However, the request to the directory node to
sign the certificate must tunnel through the onion route, so even the directory node doesn't know the IP address of the
Provider Node. The certificate contains the resource's ID, the resource's public key, and the resource's name. The
certificate is then stored on the DHT, like the node certificates.

The protocol to access the hidden service must also be specified. This could be HTTP, HTTPS, FTP, or any other protocol
that can be accessed over the internet. The protocol is stored with the resource's certificate on the DHT. Each protocol
has a customized adapter that handles the protocol-specific requests and responses, inheriting from the abstract
protocol class.

## Broker Nodes

The Provider Node must remain anonymous from any Accessor Node trying to access it. This is achieved by the use of
Broker Nodes. The Broker Nodes for a resource are nodes whose IDs are closest to the resource identifier. Any Accessor
Node who wants to access a resource can independently determine which nodes are acting as Broker Nodes for the resource,
and can contact any of them to get the resource. This ensures that the Accessor and Provider Nodes remain anonymous from
one another.

## Hosting a Hidden Service

When a Provider Node hosts a resource, they first set up an onion route. They then determine a fixed number of Broker
Nodes for the resource. The Provider Node then sends an advertisement, using the Broker Node Protocol, to the Broker
Node. The request will include the resource's certificate, which contains the resource's ID public key.

## Accessing a Hidden Service

When an Accessor Node wants to get a resource from a Provider Node, they first get the resource's certificate from the
DHT. The Accessor Node then encapsulates a request using the protocol of the resource (such as an HTTP GET for HTTPS),
inside a Broker Node Protocol request. The Broker Node will receive the request, remove the outer layer, and forward the
request to the Provider Node. The Provider Node then sends the resource back to the client via the broker node. The
Provider Node must always attach a signed hash of the resource being transferred, to ensure the resource is authentic (
the public key used is the resource's public key, stored with its certificate).

If the Hidden Service is offline, ie the provider node is offline, the broker node will return a "Service Unavailable"
response to the client (a Broker Node Protocol response). The provider node will know that the service is offline, as it
won't know any exit nodes that map to the resource identifier.

## Routing

![HiddenServiceRoute](HiddenServiceRoute.png)