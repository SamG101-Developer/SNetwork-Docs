# Routing Layer

The Routing Layer (Level 2) is the layer that sets up onion routes, manages tunneling, and reroutes traffic. The
connection protocol is a slightly varied extension of the `Layer 4` connection protocol. The key difference is that the
route node cannot authenticate the client, because the route node cannot know who the client is. This means that the
client cannot sign its ephemeral keys being sent to the relay node. Instead, it send unsigned ephemeral keys, and the
route node sends a signed hash of the keys back to the node. This way, the client can verify that the relay node has
received the genuine key, without revealing its identity.

## Protocol

In the following example, `Node A` wants to connect to `Node X` via a partial route already established. All messages
will tunnel back and forth along the partial route. See the [connection layer](Connection-Layer.md#protocol) for
descriptions on notation.

1. `NodeA` connection request
    - Get route token `T` (256-bit random || 64-bit fixed timestamp)
    - Generate an ephemeral asymmetric key pair: `(tPKa, tSKa)` unique to this connection
    - Create a JSON request `R` with `ConnectionRouteJoinRequest(T, tPKa)`
    - Send `R` to `Node X`

2. `Node X` handles the connection request `ConnectionRouteJoinRequest(T, tPKa)`
    - Validate the timestamp part of `T` and the byte form of `tPKa`
    - Generate a master session key `K` from a `CSPRNG`
    - KEM `K` with `tPKa` to create `E`
    - Sign `T || tPKa` with `sSKb` to create `S1`
    - Sign `T || E` with `sSKb` to create `S2`
    - Create a JSON response `R` with `ConnectionAccept(T, E, S1, S2)`
    - Send `R` to `Node A`

3. `Node A` handles the connection accept `ConnectionRouteJoinAccept(T, E, S1, S2)`
    - Verify `S1` with `sPKb` to match `T || ePKa`
    - Verify `S2` with `sPKb` to match `T || E`
    - Decrypt `E` with `eSKa` to get `K`
    - Save the key `K` and use KDF to generate `EK`
    - Discard the ephemeral key pair `(ePKa, eSKa)`
    - Encrypted tunnel over `EK` is established
