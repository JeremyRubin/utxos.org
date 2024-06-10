---
title: "Bitcoin Clique"
date: 2024-06-10T10:45:58-07:00
author: "Siavash Riahi, Orfeas Stefanos Thyfronitis Litos"
---

Blockchains suffer from scalability limitations, both in terms of latency and
throughput. Various approaches to alleviate this have been proposed, most
prominent of which are payment and state channels, sidechains, commit-chains,
rollups, and sharding. This work puts forth a novel commit-chain protocol,
Bitcoin Clique. It is the first trustless commit-chain that is compatible with
all major blockchains, including (an upcoming version of) Bitcoin.

Clique enables a pool of users to pay each other off-chain, i.e., without
interacting with the blockchain, thus sidestepping its bottlenecks. A user can
directly send its coins to any other user in the Clique: In contrast to payment
channels, its funds are not tied to a specific counterparty, avoiding the need
for multi-hop payments. An untrusted operator facilitates payments by
verifiably recording them.

Furthermore, we define and construct a novel primitive, Two-Shot Adaptor
Signatures, which is needed for Bitcoin Clique while being of independent
interest. This primitive extends the functionality of normal Adaptor Signatures
by allowing the extraction of the witness only after two signatures are
published on the blockchain.

Continue Reading: [https://eprint.iacr.org/2024/025](https://eprint.iacr.org/2024/025)
