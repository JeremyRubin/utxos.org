---
title: "Implementations"
date: 2019-09-27T21:56:54-07:00
---

There are a couple different implementations and deployment strategies for OP_CHECKTEMPLATEVERIFY:

* `OP_NOP4` Softfork
* Taproot Tapscript Extension 
* Templated Tapscript Extension

This page serves to mark the differences between these versions.

OP_CHECKTEMPLATEVERIFY is in some places (e.g., archives) referred to as OP_SECURETHEBAG or
OP_CHECKOUTPUTSHASHVERIFY as working names for the technique while under development while under
development.

## OP_NOP4 Softfork

This is currently the favored implementation and deployment of `OP_CHECKTEMPLATEVERIFY`.

In this version, `OP_CHECKTEMPLATEVERIFY` is implemented as a soft-fork upgrade to `OP_NOP` much like
`OP_CHECKLOCKTIMEVERIFY`. This would make `OP_CHECKTEMPLATEVERIFY` available for both SegWit and plain
script outputs.

<i>Note: P2SH is not compatible with `OP_CHECKTEMPLATEVERIFY` inehrently because of the hash cycle caused
by putting the redeemScript on the scriptSig, which changes the TXID.</i>

The implementation suggests to begin signaling for the soft-fork on versionbit 25 on January 1st,
2020 for 1 year, but this can be replaced with actual values once formally proposed.


### Links
* [BIP Draft](https://github.com/JeremyRubin/bips/blob/op-secure-the-bag-master/bip-secure-the-bag.mediawiki)
* [Implementation](https://github.com/JeremyRubin/bitcoin/tree/securethebag_master)

## Taproot Tapscript Extension

This version of `OP_CHECKTEMPLATEVERIFY` builds on Tapscript's proposed `OP_SUCCESS{X}` script upgrade
mechanism. Therefore `OP_CHECKTEMPLATEVERIFY` is only available within Tapscript.

Deployment would only be possible after or with Taproot's deployment.

Given the uncertainty with Taproot's deployment, the `OP_NOP4` deployment strategy has been
drafted.

### Links
* [BIP Draft](https://github.com/JeremyRubin/bips/blob/op-secure-the-bag/bip-secure-the-bag.mediawiki)
* [Implementation](https://github.com/JeremyRubin/bitcoin/tree/secure_the_bag)

## Templated Tapscript Extension

Because of the structure of Taproot, an `OP_CHECKTEMPLATEVERIFY` script has an
additional 32 byte overhead compared to a bare `OP_NOP4` style upgrade. By
templating the Tapscripts to support a special case `OP_CHECKTEMPLATEVERIFY`, this
overhead can be eliminated.

This technique is notable because it could be added to Taproot after the
`OP_NOP4` upgrade, but cannot be added to Taproot as a soft-fork if it is not
natively supported in Tapscript.

### Links
[Implementation](https://github.com/JeremyRubin/bitcoin/tree/taproot-with-builtin-templates)

---------------

In addition, there is a withdrawn version, `OP_CHECKOUTPUTSHASHVERIFY`, which was vulnerable to
malleability issues.
