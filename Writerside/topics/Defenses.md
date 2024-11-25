# Defenses

#### End to End Traffic Correlation

End to end traffic correlation is a method of identifying the source and destination of a message by observing the
traffic at both ends of the communication. For example, seeing connections to a server being opened at the same time
traffic to known relay nodes is being sent from a device. This can be used to prove, with a high likelihood, that the
device is the source of the traffic.

**Mitigation 1**: Dummy packets are crafted and sent from the device to relay nodes, and from relay nodes back to the
device. This bidirectional injection of dummy packets alters the network's timing and volume flow, making it harder to
correlate the traffic. The dummy packets will have a random realistic payload size, and will be released at random
intervals. They will mimic actual traffic patterns, such as HTTP GET requests.

**Mitigation 2**: Valid packets will have their traffic "padded" by the relay nodes after a layer of encryption is
removed. This is to prevent the known byte-reduction in size after encryption is removed from being used to correlate
traffic. The padding will be random bytes and of a random bounded length, and will be removed by the next relay nodes
before the next layer of encryption is unwrapped.

**Mitigation 3**: Timing injection on at each relay node means that packets will be delayed by a small, random amount of
time. This means that the time taken for a message to travel through the network is not constant. Packet reordering
doesn't matter because at the network level they have mitigations built in for this, like sequence numbers.

**Mitigation 4**: The architecture of the network is fully distributed, rather than a large fixed selection of relay
nodes being available. This means that there are no "known relay nodes", and correlating traffic going into known nodes
becomes impossible, as every network node is a relay node. The DHT is used as a PKI, ensuring authenticated handshakes
provide authenticated encrypted connections.

Overall, instead of messages being sent with known gaps and being received by some server with the same gaps, the
message timing will be completely different. Dummy packets will be injected that not only change the network timing
flow, but also the network volume flow.

#### DDoS Attacks

The only viable DDoS attack to the network is against the directory service nodes. This is because there aren't any
fixed relay nodes, and the network is fully distributed. The directory service nodes are the only nodes that are
statically known, and are the only nodes that can be targeted. If they go offline, the network can continue operated on
its own, but new nodes can't join the network, as they need certificates.

**Mitigation 1**: There are several directory service nodes, and they are all geographically distributed. This means
that there is not a single point of failure, and that the network can continue to operate if one or more of the
directory service nodes go offline.

**Mitigation 2**: Anycasting is used to route traffic to the directory service nodes. 