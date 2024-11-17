# Connection Layer 3

The Connection Layer (Level 4) is the deepest layer of the Communication Stack. It is the only layer that doesn't use
encryption, as it is the layer that establishes the encrypted connection between 2 clients. A multi-message protocol is
used to authenticate the clients and establish the connection.

## Protocol

In the following example, `Node A` wants to connect to `Node B`. Nodes all have static key pairs, and their static
public keys are hosted on the distributed public key infrastructure, embedded in certificates tied to their IDs. For
example, the static public key of `Node A` is `sPKa`.

1. `NodeA` connection request
    - Generate a random connection token `T` (256-bit random || 64-bit timestamp)
    - Generate an ephemeral asymmetric key pair: `(ePKa, eSKa)` unique to this connection
    - Sign `T || ePKa` with `sSKa` to create `S1`
    - Create a JSON request `R` with `ConnectionRequest(T, ePKa, S1)`
    - Send `R` to `Node B`

2. `Node B` handles the connection request `ConnectionRequest(T, ePKa, S1)`
    - Verify `S1` with `sPKa` to match `T || ePKa`
    - Validate the timestamp part of `T` and the byte form of `ePKa`
    - Generate a master session key `K` from a `CSPRNG`
    - KEM `K` with `ePKa` to create `E`
    - Sign `T || E` with `sSKb` to create `S2`
    - Create a JSON response `R` with `ConnectionAccept(T, E, S2)`
    - Send `R` to `Node A`

3. `Node A` handles the connection accept `ConnectionAccept(T, E, S2)`
    - Verify `S2` with `sPKb` to match `T' || E`
    - Verify `T'` matches `T'`
    - Decrypt `E` with `eSKa` to get `K`
    - Save the key `K` and use KDF to generate `EK`
    - Discard the ephemeral key pair `(ePKa, eSKa)`
    - Encrypted channel over `EK` is established

4. `Node A` key refresh (every 10 minutes)
    - Generate a new master key `K'` from a `CSPRNG`
    - Encrypt `K || K'` using the `AES_KEYWRAP` algorithm with `EK` to create `E`
    - Increment `n` (number of key refreshes)
    - Choose `m` - a future message number
    - Create a JSON request `R` with `KeyRefresh(E, n, m)`
    - Encrypt `R` with `EK` to create `ER` (current encrypted channel encryption).
    - Send `ER` to `Node B`

5. `Node B` handles the key refresh `KeyRefresh(EK, EK', n)`
    - Decrypt `E` with `EK` to get `K || K'`
    - Verify `n` is greater than the last key refresh
    - Verify `K` matches the current master key
    - Save the new master key `K'` and use KDF to generate `EK'`
    - After message `m` is received, switch to `EK'` for the encrypted channel
    - Discard the old master key `K`

**Notes:**
- All signatures are timestamped and contain the session token and target identifier.
- Memory is zeroed after use to prevent memory leaks, and all memory operations are constant time.
- Any signature/certificate verification failure (including stale timestamps) results in the connection being closed.
- Mutexes lock the caches before read/write operations.
- Caches are LRU and bounded to prevent memory exhaustion attacks.
- Connection rate limiter
- Node state is used in conjunction with message types to prevent replay attacks.
- Algorithms:
    - Hashing: `SHA_3_256`,
    - Symmetric Encryption: `AES_256_OCB3`,
    - Asymmetric KEM: `CRYSTALS-Kyber`,
    - Asymmetric Signing: `CRYSTALS-Dilithium`,
    - Certificates: `X509`,
    - Key derivation: `HKDF (KMAC-SHA256)`,
    - MAC: `KMAC-SHA3_256`.
