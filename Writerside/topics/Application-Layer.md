# Application Layer

The Application Layer is a layer that wraps a number of substitutable internals layers, such as a HTTP layer, or FTP
layer etc. It allows for the communication stack to be used for any application, by providing a common interface for
applications to use.

## HTTP Layer

The most common layer is the HTTP Layer. This allows for internet applications such as Chrome or Edge to set the proxy
to the HTTPLayer address, and then use the HTTPLayer to send and receive HTTP requests and responses via the network.
The following diagram outlines this:
![HTTP Layer Diagram](Proxy.svg)

## Custom Applications

Whilst standardized protocols such as HTTP, used by browsers such as Chrome can link into the Application Layer, it is
also possible to utilize the DHT to create custom anonymous applications, such as a social media platform. This would
work by using the `Layer 3` extension of the application layer class, allowing for DHT interfacing.
