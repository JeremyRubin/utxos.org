---
title: "Safer Hashed Time Locked Contracts (HTLCS) Limits"
date: 2019-10-18T10:30:40-07:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---

![](/images/uses/htlcs.png)

In the Lightning Network protocol, Hashed Time Locked Contracts (HTLCS) are used
in the construction of channels. A new HTLC is required per route that the
channel is serving in.


In [BOLT #2](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md),
this maximum number of HTLCs in a channel is hard limited to 483 as the maximum
safe size to prevent the transaction from being too large to be valid. Adding
more HTLCs would interfere with being able to broadcast and confirm a
transaction.


Therefore, similarly to how congestion control is handled for normal
transaction, lightning channel updates can be done across an `OP_CHECKTEMPLATEVERIFY`
tree, allowing nodes to safely use many more HTLCS.

This has the benefit that less data would be used to close any individual HTLC
in addition to permitting more than 483, so it may be useful to use CTV even if
there are fewere than 483 HTLCs.

Because each HTLC can have its own relative time lock in the tree, this also
improves the latency senstivity of the lightning protocol on contested channel
close.

_errata: previously, this page suggested that the limit was 12 for LND, which
was incorrect (12 was the field number in the struct)._
