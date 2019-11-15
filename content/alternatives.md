---
title: "Alternatives"
date: 2019-09-28T17:48:34-07:00
---

`OP_CHECKTEMPLATEVERIFY` has a multitude of benefits for the Bitcoin ecosystem. Are
there other paths to these improvements? In short, yes. This page has a survey
of the other methods for enabling `OP_CHECKTEMPLATEVERIFY` like functionality and why
such techniques are not suitable substitutes.

## OP_CHECKOUTPUTVERIFY
[MES16](https://fc16.ifca.ai/bitcoin/papers/MES16.pdf) _presents an extension
to Bitcoin’s script language enabling covenants, a primitive that allows
transactions to restrict howthe value they transfer is used in the future._

`OP_COV` allows a user to specify a pattern and and output index. The pattern
is then evaluated against the output at that index to see if the output is a
valid representative of the set described by the pattern.

This can be used to emulate something similar to `OP_CHECKTEMPLATEVERIFY` by
specifying two covenants for outputs 0 and 1 to specify the expanding of a
payment tree.

However, there are a few drawbacks:

1. `OP_COV` does not have a mechanism to restrict the number of inputs to a
   single input, meaning the half-spend problem exists
1. A user needs to specify 2 patterns, as opposed to 1 hash with
   `OP_CHECKTEMPLATEVERIFY`
1. `OP_COV` is not designed to ensure a stable TXID for the transactions
   (e.g., locktimes could be malleated) which makes `OP_COV` based congestion
   control unsuitable for lightning channel like contracts or CPFP payments.
1. `OP_COV` covenant patterns are not "computationally enumerable". That is,
   it is not generally possible to enumerate all possible spends of an
   `OP_COV` covenant. Whereas with `OP_CHECKTEMPLATEVERIFY`, the limitations ensure
   that a spending party (by constructing the key) knows all possible
   executions.

## OP_PUSHTXDATA

A [Draft BIP](https://github.com/jl2012/bips/blob/vault/bip-0ZZZ.mediawiki) by
Johnson Lau proposes `OP_PUSHTXDATA`.

`OP_PUSHTXDATA` is a (almost) superset of `OP_CHECKTEMPLATEVERIFY` in the sense
that it is possible to encode an `OP_CHECKTEMPLATEVERIFY` into an `OP_PUSHTXDATA`.

To do so, one simply uses `OP_PUSHTXDATA` to check that the nVersion,
nLocktime, vouts, sequences, and vins match the expected values.

However, there are a few drawbacks:

1. As specified, `OP_PUSHTXDATA` does not allow the user to get the scriptSig
   of an input on the stack. This means that without amendment,
   `OP_PUSHTXDATA` cannot guarantee TXID stability and is unsuitable for
   constructing lightning channel like contracts from non-segwit outputs as the
   stack can be malleated (segwit guarantees scriptsigs are null).
1. The scripts for `OP_PUSHTXDATA` to emulate `OP_CHECKTEMPLATEVERIFY` are rather
   long in comparison as they require committing to many fields. For example:
```forth
1 OP_PUSHTXDATA 1 OP_EQUALVERIFY
2 OP_PUSHTXDATA 2 OP_EQUALVERIFY
5 OP_PUSHTXDATA 2 OP_EQUALVERIFY
6 OP_PUSHTXDATA 0 OP_EQUALVERIFY
0 11 OP_PUSHTXDATA 0 OP_EQUALVERIFY
0 15 OP_PUSHTXDATA <value vout 0> OP_EQUALVERIFY
                   <scriptPubkey vout 0> OP_EQUALVERIFY
1 15 OP_PUSHTXDATA <value vout 1> OP_EQUALVERIFY
                   <scriptPubkey vout 1> OP_EQUALVERIFY 
```
v.s.

```forth
<Template Hash> OP_CHECKTEMPLATEVERIFY 
```
`OP_PUSHTXDATA` could ameliorate this by adding an data type to push the Bag
Hash from `OP_CHECKTEMPLATEVERIFY`.
1. As a superset of functionality, `OP_PUSHTXDATA` enables a lot of new use cases
   that we may or may not want to support. `OP_CHECKTEMPLATEVERIFY` is, by
   comparison, conservative in what it enables.
1. `OP_CHECKTEMPLATEVERIFY` is an `OP_NOP` upgrade, which maintains compatibility
   with old scripts, whereas `OP_PUSHTXDATA` requires new execution semantics.
   `OP_PUSHTXDATA` could be made to work with soft-fork friendly VERIFY
   semantics, but it would be unwieldy.


## OP_CAT + OP_CHECKSIGFROMSTACKVERIFY

By enabling `OP_CHECKSIGFROMSTACKVERIFY` and `OP_CAT` it [would be
possible](https://blockstream.com/2016/11/02/en-covenants-in-elements-alpha/) to
enable covenants. Russel O'Connor
[proposes](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-May/016946.html)
to do this instead of `OP_CHECKTEMPLATEVERIFY` or `ANYPREVOUT`.

Essentially what `OP_CHECKSIGFROMSTACKVERIFY` and `OP_CAT` lets us do is
emulate and OpCode via a script gadget which results in the signature message
being on the stack, i.e., like `OP_PUSHTXDATA`, but all the data gets pushed
as one serialized blob. It's then possible to check that this value matches a
pre-supposed template (bits like the prevouts can be passed in through the
witness data).

However, there are a few drawbacks:

1. This form of covenants is possible, but highly complex to understand and
   implement without higher-order scripting primitives.
1. Further, with extensions like `OP_EC_KEY_TWEAK`, it would be possible to
   enable recursive covenants. That means this strategy might limit future
   extensions to Bitcoin with unintended "constructive interference"
   capabilities.
1. If the functionality provided is roughly equivalent to `OP_PUSHTXDATA`, it
   would be easier to just to implement `OP_PUSHTXDATA`
1. Validation is more expensive to use the `OP_CHECKSIGFROMSTACK` gadget
   compared to `OP_CHECKTEMPLATEVERIFY` or `OP_PUSHTXDATA`.
1. Desire to disable PubKey recovery for signatures would disable the recursive
   use of this type of covenant, which would make the technique not able to
   be used for `OP_CHECKTEMPLATEVERIFY` type use.



## OP_CHECKTXOUTSCRIPTHASHVERIFY

`OP_CHECKTXOUTSCRIPTHASHVERIFY`, proposed on the [mailing
list](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-October/016448.html)
is similar to `OP_CHECKTEMPLATEVERIFY`, but only a single output script is committed
to and not the amount of Bitcoin to be paid to that output. The stated design
goal is theft-resistant vaults.

As specified, `OP_CHECKTXOUTSCRIPTHASHVERIFY` is not even sufficient for its
own purported use case:

1. Inability to limit the amount of fee paid, ensuring the amount of value
   forwarded
1. Inability to limit the number of outputs created

And is insufficient for `OP_CHECKTEMPLATEVERIFY` semantics for a myriad of reasons
more.


## SIGHASH_NOINPUT / ANYPREVOUT

With `SIGHASH_NOINPUT`, it should be theoretically possible for a bare script
(non-segwit) to be specified as follows to emulate `OP_CHECKTEMPLATEVERIFY`:

```forth
scriptSig: <random sig || SIGHASH_NOINPUT>
scriptPubkey: <pkR> OP_CODESEPARATOR OP_CHECKSIGVERIFY 
```

It would then be possible to compute the `pkR` as being the recovered pubkey
from the transaction without signing the inputs, effectively committing to the
information in the same manner as `OP_CHECKTEMPLATEVERIFY`.

In a segwit type transaction, this is not possible (without additional
modifications) because the scriptPubkey always commits to the entire pubkey,
preventing pubkey recovery techniques.

`SIGHASH_NOINPUT` is furthermore insufficient to emulate `OP_CHECKTEMPLATEVERIFY`
as the TXID can be malleated by modifying the scriptSig(s) in the transaction,
which makes it unsuitable for use in Lightning Channel construction or CPFP
transactions.




### Acknowledgements

Thanks to Olaolu Osuntokun and Bob McElrath whose talks helped to inform this
survey.

* https://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2019-06-06-noinput-etc/
* https://diyhpl.us/wiki/transcripts/2019-02-09-mcelrath-on-chain-defense-in-depth/
