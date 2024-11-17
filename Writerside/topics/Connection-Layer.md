# Connection Layer

The Connection Layer (Level 4) is the deepest layer of the Communication Stack. It is the only layer that doesn't use
encryption, as it is the layer that establishes the encrypted connection between 2 clients. A multi-message protocol is
used to authenticate the clients and establish the connection.

## Protocol

In the following example, `Node A` wants to connect to `Node B`. Nodes all have static key pairs, and their static
public keys are hosted on the distributed public key infrastructure, embedded in certificates tied to their IDs. For
example, the static public key of `Node A` is `sPKa`.

1. `Node A` connection preparation
   - Generate a random connection token `T` (24-bit random || 8-bit timestamp)
   - Cache the connection token `T`
   - Generate an ephemeral asymmetric key pair: `(ePKa, eSKa)` unique to this connection
   - Sign `T || ePKa` with `sSKa` to create `S1`
   - Create a JSON request `R` with `(ConnectionRequest, T, Cert_A, ePKa, S1)`
   - Send `R` to `Node B`

2. `Node B` handles the connection request `(ConnectionRequest, T, Cert_A, ePKa, S1)`
   - Verify `Cert_A` with a hard-coded directory service public key
   - Verify `S1` with `sPKa` (from `Cert_A`) to match `T || ePKa`
   - Verify the timestamp in `T` to be within a set tolerance
   - Cache the connection token `T`
   - Create a random challenge `C` (120-bit random || 8-bit timestamp)
   - Cache the challenge `C`
   - Sign `T || C` with `sSKb` to create `S2`
   - Create a JSON response `R` with `(SignatureChallenge, T, C, S2)`
   - Send `R` to `Node A`

3. `Node A` handles the connection response `(SignatureChallenge, T, C, S2)`
   - Verify `S2` with `sPKb` to match `T || C`
   - Verify `T` matches the cached connection token
   - Verify the timestamp in `C` to be within a set tolerance
   - Cache the challenge `C`
   - Sign `T || C` with `eSKa` to create `S3`
   - Create a JSON request `R` with `(ChallengeResponse, T, S3)`
   - Send `R` to `Node B`

4. `Node B` handles the challenge response `(ChallengeResponse, T, S3)`
   - Verify `S3` with `ePKa` to match locally stored `C`
   - Create a 512-bit cryptographically secure pseudo-random key `K`
   - Sign `K` with `sSKb` to create `S4`
   - Encapsulate `T || K || S4` with `ePKa` to create `E`
   - Create a JSON response `R` with `(ConnectionAccept, T, E)`
   - Send `R` to `Node A`

5. `Node A` handles the connection accept `(ConnectionAccept, T, E)`
   - Decrypt `E` with `eSKa` to get `T || K || S4`
   - Verify `S4` with `sPKb` to match `K`
   - Verify `T` matches the cached connection token
   - Save the key `K`
   - Hash the key `K` to create `H` (for connection confirmation)
   - Sign `H` with `eSKa` to create `S5`
   - Create a JSON request `R` with `(ConnectionConfirm, H, S5, T)`

6. `Node B` handles the connection confirm `(ConnectionConfirm, H, S5, T)`
   - Verify `S5` with `ePKa` to match `H`
   - Verify `T` matches the cached connection token
   - Hash the key `K` to create `G`
   - Check `G` against `H` to prove ownership from `Node A`

7. Both `Node A` and `Node B` can now communicate using the shared key `K`
   - Both nodes use a KDF algorithm to derive an encryption key `EK` from `K`

**Notes:**
- All random information is generated from cryptographically secure pseudo-random number generators.
- All signatures are timestamped and contain the target identifier, to prevent replay attacks.
- The ephemeral key pair is unique to this connection session and discarded after the connection is established.
- Any signature/certificate verification failure (including stale timestamps) results in the connection being closed.
- Mutexes lock the caches before read/write operations.
- Caches have items purged after `n` seconds, which is the timestamp tolerance value.
- Algorithms: `SHA_3_256`, `AES_256_OCB3`, `RSA_2048`, `X509`, `HKDF (KMAC-SHA256)`.
