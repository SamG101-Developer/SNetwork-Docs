# Connection Layer

The Connection Layer (Level 4) is the deepest layer of the Communication Stack. It is the only layer that doesn't use
encryption, as it is the layer that establishes the encrypted connection between 2 clients. A multi-message protocol is
used to authenticate the clients and establish the connection.

## Protocol

In the following example, `Node A` wants to connect to `Node B`. Nodes all have static key pairs, and their static
public keys are hosted on the distributed public key infrastructure, embedded in certificates tied to their IDs. For
example, the static public key of `Node A` is `sPKa`.

1. `Node A` connection preparation
    - Generate a random connection token `T`
    - Generate an ephemeral asymmetric key pair: `(ePKa, eSKa)`
    - Sign `ePKa` with `sSKa` to create `S1`
    - Create a JSON request `R` with `(ConnectionRequest, T, Cert_A, ePKa, S1)`
    - Send `R` to `Node B`

2. `Node B` handles the connection request `(ConnectionRequest, T, Cert_A, ePKa, S1)`
    - Verify `S1` with `sPKa` (from `Cert_A`) to match `ePKa`
    - Create a random challenge `C` (24-bit random || 8-bit timestamp)
    - Create a JSON response `R` with `(SignatureChallenge, T, C)`
    - Send `R` to `Node A`
    - **Note that `C` doesn't have to be signed**

3. `Node A` handles the connection response `(SignatureChallenge, T, C)`
    - Verify the timestamp in `C` to be within a set tolerance
    - Save the challenge `C`
    - Sign the challenge with `eSKa` to create `S2`
    - Create a JSON request `R` with `(ChallengeResponse, T, S2)`
    - Send `R` to `Node B`

4. `Node B` handles the challenge response `(ChallengeResponse, T, S2)`
    - Verify `S2` with `ePKa` to match locally stored `C`
    - Create a 32-bit cryptographically secure pseudo-random key `K`
    - Sign `K` with `sSKb` to create `S3`
    - Encapsulate `K || S3` with `ePKa` to create `E`
    - Create a JSON response `R` with `(ConnectionAccept, T, E)`
    - Send `R` to `Node A`
    - **Because comparison is against locally stored `C`, signature is not required**

5. `Node A` handles the connection accept `(ConnectionAccept, T, E)`
    - Decrypt `E` with `eSKa` to get `K || S3`
    - Verify `S3` with `sPKb` to match `K`
    - Save the key `K`
    - Connection is established

6. Both `Node A` and `Node B` can now communicate using the shared key `K`
    - Both nodes use a KDF algorithm to derive an encryption key `EK` from `K`
