---
title: "Increasing Max Hashed Time Locked Contracts (HTLCS)"
date: 2019-10-18T10:30:40-07:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---

![](/images/uses/htlcs.png)

In the Lightning Network protocol, Hashed Time Locked Contracts (HTLCS) are used
in the construction of channels. A new HTLC is required per route that the
channel is serving in.


In [BOLT #2](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md),
this maximum number of HTLCs in a channel is hard limited to 483 as the maximum
safe size to prevent the transaction from being too large to be valid. In common
software implementations such as LND, this limit is set much lower to [12
HTLCS](https://github.com/lightningnetwork/lnd/blob/21a40daf5840a856240866fff49e8c07dac7283c/lnrpc/rpc.proto#L963).
This is because accepting a larger number of HTLCS makes it more difficult for
transactions to confirm during congested periods as they must pay higher fees.

Therefore, similarly to how congestion control is handled for normal
transaction, lightning channel updates can be done across an `OP_CHECKTEMPLATEVERIFY`
tree, allowing nodes to safely use many more HTLCS.

Because each HTLC can have its own relative time lock in the tree, this also
improves the latency senstivity of the lightning protocol on contested channel
close.
