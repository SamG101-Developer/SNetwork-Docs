# Connection Layer 2

The Connection Layer (Level 4) is the deepest layer of the Communication Stack. It is the only layer that doesn't use
encryption, as it is the layer that establishes the encrypted connection between 2 clients. A multi-message protocol is
used to authenticate the clients and establish the connection.

## Protocol

In the following example, `Node A` wants to connect to `Node B`. Nodes all have static key pairs, and their static
public keys are hosted on the distributed public key infrastructure, embedded in certificates tied to their IDs. For
example, the static public key of `Node A` is `sPKa`.

1. `Node A` connection preparation
   - Generate a random connection token `T` (256-bit random || 16-bit counter || 64-bit ms timestamp - epoch)
   - Cache the connection token `T`

2. `Node A` handshake request
   - Generate an ephemeral asymmetric key pair: `(ePKa, eSKa)` unique to this token
   - Sign `T || ePKa` with `sSKa` to create `S1`
   - Create a JSON request `R` with `(ConnectionHandshakeRequest, T, ePKa, S1)`
   - Send `R` to `Node B`
   - Thread: Get `sPKb` from the certificate of `Node B` on the DHT PKI

3. `Node B` connection preparation
   - Validate `T` to be within a set tolerance
   - Validate the counter hasn't been seen from `Node A` before

4. `Node B` handshake response
   - Generate an ephemeral asymmetric key pair: `(ePKb, eSKb)` unique to this token
   - Sign `T || ePKb` with `sSKb` to create `S2`
   - Create a JSON response `R` with `(ConnectionHandshakeResponse, T, ePKb, S2)`
   - Send `R` to `Node A`
   - Thread: Get `sPKa` from the certificate of `Node A` on the DHT PKI

5. Nodes derive the shared secret
   1. `Node A` derives the shared secret   
      - Verify `S2` with `sPKb` to match `T || ePKb`
      - Combine `ePKb` with `eSKa` to derive the shared secret `S`
      - Hash `S` to create `H`
      - Sign `T || H` with `sSKa` to create `S3`
      - Create a JSON request `R` with `(ConnectionHandshakeConfirm, T, S3)`

   2. `Node B` derives the shared secret
      - Verify `S1` with `sPKa` to match `T || ePKa`
      - Combine `ePKa` with `eSKb` to derive the shared secret `S`
      - Hash `S` to create `H`
      - Sign `T || H` with `sSKb` to create `S4`
      - Create a JSON response `R` with `(ConnectionHandshakeConfirm, T, S4)`

6. Connection confirmation
   1. `Node A` confirms the connection
      - Verify `S4` with `sPKb` to match `T || H`
      - KDF `S` to derive the session key `E` (for AES-256-OCB3)
   2. `Node B` confirms the connection
      - Verify `S3` with `sPKa` to match `T || H`
      - KDF `S` to derive the session key `E` (for AES-256-OCB3)


**Notes:**
- All random information is generated from cryptographically secure pseudo-random number generators.
- All signatures are timestamped and contain the target identifier, to prevent replay attacks.
- The ephemeral key pair is unique to this connection session and discarded after the connection is established.
- Memory is zeroed after use to prevent memory leaks.
- Any signature/certificate verification failure (including stale timestamps) results in the connection being closed.
- Mutexes lock the caches before read/write operations.
- Caches are LRU and bounded to prevent memory exhaustion attacks.
- Algorithms: `SHA_3_256`, `AES_256_OCB3`, `Curve25519`, `X509`, `HKDF (KMAC-SHA256)`.
- All comparison operations are done in constant time to prevent timing attacks.
- Connection rate limiter