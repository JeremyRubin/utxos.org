---
title: "Scaling"
date: 2019-10-10T10:45:58-07:00
weight: 1
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---
![](/images/uses/scalepay.svg)

When there is a high demand for blockspace it becomes very expensive to make
transactions. By using OP_CHECKTEMPLATEVERIFY, a large volume payment processor may
aggregate all their payments into a single O(1) transaction for purposes of
confirmation. Then, some time later, the payments can be expanded out of that
UTXO when the demand for blockspace is decreased.

Without OP_CHECKTEMPLATEVERIFY, this is still possible to do with Schnorr signatures
(even with ECDSA given multiparty schemes). However, it is not possible to do
non-interactively, which fundamentally limits the viability of the approach as
interacting to collect signatures from all recipients may be difficult or slow.

The sender of a congestion controlled transaction can choose from many different
transaction structures.

The simplest option is a single committed transaction which expands from 1
output to N. Even this simple use case is highly useful because the expansion
can wait till cheap block space is available.

As the number of recipients grows, a sender can commit to a tree of outputs
using OP_CHECKTEMPLATEVERIFY. The tree permits them to confirm as many payments as they
like (i.e., even more than can fit into a block). Such a technique starts to
make sense when:

`N*size_of(Output) > log(N)*size_of(OP_CHECKTEMPLATEVERIFY Txn) + size_of(Output)`

Assuming straightforward transaction sizes, this is around 10
recipients.

Furthermore, the script can commit to variable size expansions -- say, one node
which expands by 2, by 4, by 8, etc. This allows a trade off between transaction
overhead and immediately available block space. Depending on implementation
(either at the transaction level, the script level, or via Taproot), the Merkle
tree lookup in that case is O(log(log(N))) extra overhead, but the tree can be
Huffman encoded to make it E[O(1)] depending on block demand.

Each node of the tree can also attempt to 'opt in' to preferring a multiparty
signature based spend (using Schnorr signatures or Taproot), but if participants
are offline or malicious the expansion can proceed to smaller groups.

The overall overhead of the tree approach (without optimizations) is from the
perspective of each user O(log(N)) transactions with an expectation of just 1
additional transaction, and 2N from the perspective of the network. However,
given the lack of signatures required for such transactions, the actual overhead
is less.


Below is an example implementation of a tree payment in [sapio](https://learn.sapio-lang.org).

{{% code file="/static/sapio/sapio-contrib/src/contracts/treepay.rs" language="rust" %}}
