# Reputation System

> Assume a route from `Node A` through entry node `Node X` and attempted intermediary node `Node Y`.

The DHT utilizes a "shared punishment" reputation system to ensure that nodes are not malicious. This mitigates attacks
where nodes report other nodes as offline, in order to force routes through certain nodes. It is not possible to tell
whether `Node X` is lying about `Node Y` being offline, or if `Node Y` pretended to be offline when `Node X` requested a
connection. For example, if `Node X` reports `Node Y` as offline, `Node A` could connect and determine `Node Y` to be
online, but whether `Node X` lied or `Node Y` rejected `Node X` is unknown.

Whilst consensus mechanisms were considered, they were rejected because providing proof of not connecting isn't
possible. For example, `Node X` could try and fail to connect to `Node Y`, and if the consensus determines `Node Y` is
online, did `Node X` ever make a connection request? This is validatable, but did `Node Y` ever respond? This is
impossible to determine, because if `Node Y` wanted `Node X` punished, they would say they never received the request,
and if `Node X` wanted `Node Y` punished, they would pretend to have never received `Node Y`'s response.

Therefore, a shared punishment system was implemented, which penalizes both `Node X` and `Node Y` if `Node X` reports
`Node Y` as offline to `Node A`. This means that if `Node X` was mas reporting other nodes as offline, to force routing
through a malicious node friendly to `Node X`, `Node X` would be heavily penalized, and the other nodes would only
lightly penalized. The same is for `Node Y`; if `Node Y` kept pretending to be offline in different circuits, to try and
reduce other node's reputation, it would end up being heavily penalized.

The next issue is the DHT having to trust `Node A` that `Node X` and `Node Y` need penalties. This can be done by
providing:
- `Node A`'s request to `Node X` to extend the route to `Node Y` (signed by `Node A`)
- `Node X`'s response that `Node Y` is offline (signed by `Node X`)

This combination cannot be forged by `Node A`, as `Node X`'s response must be signed. Embedded in route messages is the
route token, meaning that `Node A` cannot reuse an old `Node X` response to penalize `Node X` and `Node Y` again. The
only other issue is `Node A` constantly requesting penalties for the same route, but this is mitigated by the DHT only
accepting penalties with unique route tokens.

## Penalty System

### Reputation of Other Nodes

An additional feature is penalty weighting. If `Node A` and `Node X` are malicious, they could try to mass create
reports against other `Node Y`s. This would destroy the reputation of their friendly `Node X`. However, the penalty to
`Node Y` is based of the reputation of `Node X`, and vice-versa. This means that if `Node X` is constantly involved in
penalties, random `Node Y`s won't suffer as much.

Further to this, `Node A`'s reputation is also taken into account. If `Node A` has a low reputation, the penalty is less
severe. This is because they are a low-trust node anyway, and whilst their penalty requests must be cryptographically
verifiable, if they collude with a malicious `Node X`, they can try to mass create reports against other `Node Y`s.

The reputations of `Node A`, `Node X` and `Node Y` are represented by $R_A$, $R_X$ and $R_Y$ respectively.

### History of Penalties

The longer a node has been behaving well, the less severe the penalty. Whilst this is also reflected in the reputation
of the well-behaved node, if it is temporarily misbehaving (being offline), it shouldn't be penalized as heavily as a
node that is constantly misbehaving. This mitigates network issues, for example, that are only temporary and
non-malicious. The history factor is represented by $H$.

It is determined by the number of penalties a node has received in the last 24 hours, and the number of penalties a node
has received in the last 7 days. The penalty is reduced by a factor of the number of penalties in the last 24 hours, and
the number of penalties in the last 7 days. This means that if a node is constantly misbehaving, the penalty is more
severe.

### Activity of Reporting

**TODO**

### Corroboration of Reports

**TODO**

### Natural Decay

Inactive nodes naturally decay over time as they are not engaged in forwarding traffic or any other beneficial activity.
This means that if a node is offline for a long time, it will naturally have a lower reputation, and won't be chosen as
often for routing.

### Formula

This creates formulas:
- $$P_X = f(R_A, R_X, R_Y, ...)$$
- $$P_Y = f(R_A, R_X, R_Y, ...)$$

The formula $f$ is a function that must apply a penalty depending on node reputations, whilst not allowing a node to
dominate the penalty system.

$$
\begin{equation}
P_\beta = f(R_A, R_\alpha, R_\beta, P_{24}, P_7) = B * R_\beta * w_A * w_\alpha * H * A
\end{equation}
$$

$$
\begin{equation}
w_A = \frac{R_A}{R_A + R_\alpha + R_\beta + \epsilon}
\end{equation}
$$

$$
\begin{equation}
w_\alpha = \frac{R_\alpha}{R_A + R_\alpha + R_\beta + \epsilon}
\end{equation}
$$

$$
\begin{equation}
H = \frac{P_{24}}{P_{24} + P_{7} + \epsilon}
\end{equation}
$$

$$
\begin{equation}
B = baseline\_constant
\end{equation}
$$

$$
\begin{equation}
\epsilon = 0.0001
\end{equation}
$$

### Properties
- **Balances impact of reputation**: The penalty is balanced between the two nodes, based on their reputation. If
  `Node X` has a high reputation, the penalty to `Node Y` is higher, and vice-versa.
- **Self-limiting collusion**: Collusion cannot effectively damage node reputations, as the penalty is based on the
  reputation of the reporting node, as their `Node X` reputation continuously decreases.
- **Low reputation nodes**: Low reputation nodes have less impact on the penalty system, as they are low-trust nodes
  anyway. This means that if they try to mass create reports, the penalty is less severe.
- **Honest behaviour encouragement**: Honest behaviour is incentivised, otherwise they will get penalized heavily for
  making continuous false reports.

## Reward System

Whilst nodes can be penalised for misbehaving, they can also be rewarded for good behaviour. Route owners will reward
participants in the route for successfully forwarding traffic. This is to incentivise nodes to participate in the
network and forward traffic.

## How Reputation is Delivered

When a route owner wants to penalise or reward a node, they generate a "token". Red tokens are for penalties, and green
tokens are for rewards. Tokens are published by the route owner to the DHT, and are validated at the storage nodes. A
token isn't usable or applicable until it has been verified by a storage node.

Storage nodes have the ability to discard invalid tokens, and tokens that have expired. There is a lightweight proof of
work system in place to prevent token spamming, and to ensure that the route owner has to put in some effort to generate
a token.

There are a few different tokens producible, and signed by the route owner. The tokens are:

<format color="Red">Malicious Route Node</format>
:
A red token that is generated for 2 nodes in a route where their connection has failed. This is to reduce the chance
of malicious nodes being chosen.
- `Node A`'s request to `Node X` to extend the route to `Node Y` (signed by `Node A`)
- `Node X`'s response that `Node Y` is offline (signed by `Node X`)
- Route token
- Timestamp & expiration
- Signature


<format color="Lime">Traffic Forwarding Route Node</format>
:
A green token that is generated for an entry node in the circuit, for participating in the route and correctly
forwarding traffic to the next node. Deemed a reliable node.
- `Node A`'s identifier
- `Node X`'s identifier
- Route token
- Timestamp & expiration
- Signature
> Because the key of this token only takes into account the identifiers and part of the timestamp, a `Node A` cannot
> token farm for malicious `Node X`'s, as the same storage nodes will just discard the token. {style="warning"}
