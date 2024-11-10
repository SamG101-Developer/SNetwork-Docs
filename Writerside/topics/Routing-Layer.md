# Routing Layer

The Routing Layer (Level 2) is the layer that sets up onion routes, manages tunneling, and reroutes traffic. There is a
complex protocol that is used to exchange cryptography keys between the client and other nodes, to ensure that the
client remains anonymous from the target node, whilst still having an authenticated connection.

## Protocol

In the following example, `Node X` is the client node who is initiating a route. `Node A`, `B` and `C` will be the
entry, intermediary and exit nodes respectively. The example shows `Node X` tunneling a connection to `Node B`.

**To "tunnel" is to send a message to a node, via the onion route, in either direction.**

1. `Node X` tunnel initiation
    - Select `Node B` off the DHT
    - Creates a random challenge `C` (24-bit random || 8-bit timestamp)
    - Create a JSON request `R` with `(ConnectionExtend, Node B, C)`
    - Send `R` to `Node A`

2. `Node A` handles the tunnel initiation `(ConnectionExtend, Node B, C)`
    - [Establish a connection](Connection-Layer.md#protocol) with `Node B`
    - Create a JSON request `R` with `(TunnelRequest, C)`
    - Send `R` to `Node B`

3. `Node B` handles the tunnel request `(TunnelRequest, C)`
    - Create an ephemeral key pair `(tPKb, tSKb)`
    - Sign `C || tPKb` with `sSKb` to create `S0`
    - Create a JSON response `R` with `(TunnelEphemeralKey, tPKb, S0)`
    - Tunnel `R` backwards to `Node X`

4. `Node X` handles the tunneling response `(TunnelEphemeralKey, tPKb, S0)`
    - Verify `S0` with `sPKb` to match `tPKb || C`
    - Create a 32-bit cryptographically secure pseudo-random key `K`
    - Encrypt `K` with `tPKb` to create `E`
    - Create a JSON request `R` with `(TunnelPrimaryKey, E)`
    - Tunnel `R` to `Node B`

5. `Node B` handles the tunneling accept `(TunnelPrimaryKey, E)`
    - Decrypt `E` with `tSKb` to get `K`
    - Save the key `K`
    - Sign `K` with `sSKb` to create `S1`
    - Create a JSON response `R` with `(TunnelAccept, S1)`
    - Tunnel `R` to `Node X`

6. `Node X` handles the tunneling accept `(TunnelAccept, S1)`
    - Verify `S1` with `sPKb` to match `K`
    - Tunneling is established

Because the target nodes in the route cannot be aware of the client's identity, the client cannot sign keys that are
sent to the target nodes. Instead, the target nodes sign what they receive and send it back to the client. This way,
the client can verify the target node has received the genuine key, without revealing its identity.
