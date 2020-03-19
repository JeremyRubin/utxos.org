---
title: "Non Interactive Channels"
date: 2019-10-18T10:46:13-07:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---

![](/images/uses/nic.svg)

Normally, when opening a payment channel, you require participation from both
parties to the channel. This is because the Lightning Network uses pre-signed
multi-sig type transactions to ensure that a channel can always be exited by
either party, before entering.

With OP_CHECKTEMPLATEVERIFY, it's possible for a single party to construct a
channel which either party can exit from without requiring signatures from
both parties.

These payment channels can operate in one direction, paying to the channel
"listener" without need for their private key to be online.



As an example, suppose Carol had a desire to non interactively create a channel
where Alice has 0.1 BTC and Bob has 0.9 BTC.

Carol creates Transaction A paying out 1 BTC to witness program:

```
OP_IF <StandardTemplateHash(Tx B)> OP_CTV
OP_ELSE 2 <PK Bob> <PK Alice> 2 CHECKMULTISIG
OP_ENDIF
```

This corresponds to the Main Channel UTXO in the diagram.

In order to redeem funds from this channel cooperatively, Alice and Bob simply use a witness:

```
0 <signature Close State Bob> <signature Close State Alice> 0
```

which can be exchanged in counterparts at any time.


Transaction B can be used in the uncooperative case. Transaction B looks like:

```
Inputs: Main Channel UTXO
Outputs: 1 BTC to Uncooperative Close Initiated UTXO
```

Uncooperative Close Initiatied UTXO has the following script:

```
OP_IF <StandardTemplateHash(Tx C)> OP_CTV
OP_ELSE 2 <PK Bob> <PK Alice> 2 CHECKMULTISIG
OP_ENDIF
```


In order to channel spend funds from this channel, Alice and Bob simply craft a
witness for the Uncooperative Close Inititiated UTXO:

```
0 <signature Revokable HTLC State Bob> <signature Revokable HTLC State Alice> 0
```

which can be exchanged in counterparts (i.e., with signatures from either Alice
or Bob) at any time. Optionally, Alice and Bob may also create this witness
signed on the Main Channel UTXO to save fees in the cooperative-but-offline
close case.

A Revokable HTLC State transaction could look like this, but is compatible with
other protocols:

```
Inputs: Uncooperative Close Initiated UTXO
Witness Data: (see above)
Outputs: 1 BTC to HTLC Script
```

Where a potential HTLC Script (for Alice to be able to close, and Bob to contest) from [BOLT#3](https://github.com/lightningnetwork/lightning-rfc/blob/master/03-transactions.md#offered-htlc-outputs).
```
# To remote node with revocation key
OP_DUP OP_HASH160 <RIPEMD160(SHA256(revocationpubkey))> OP_EQUAL
OP_IF
    OP_CHECKSIG
OP_ELSE
    <remote_htlcpubkey> OP_SWAP OP_SIZE 32 OP_EQUAL
    OP_NOTIF
        # To local node via HTLC-timeout transaction (timelocked).
        OP_DROP 2 OP_SWAP <local_htlcpubkey> 2 OP_CHECKMULTISIG
    OP_ELSE
        # To remote node with preimage.
        OP_HASH160 <RIPEMD160(payment_hash)> OP_EQUALVERIFY
        OP_CHECKSIG
    OP_ENDIF
OP_ENDIF
```


Transaction C sets up a default closure path for the channel. This is used in
the case where either Alice or Bob never come online. Transaction C looks like
this:


```
Inputs: Uncooperative Close Initiated UTXO
Witness Data: 1
Sequence: + 2 weeks
Outputs: 0.1 BTC to Alice, 0.9 BTC to Bob
```




### Minor Security Note
Note that because the full uncooperative close path is public, it may be
possible for a mallicious actor Eve to partition both Alice and Bob from the
network and close the channel via transaction C if Eve can learn the details
of the CTV transaction.


To protect against this outcome, multisigs can be added, e.g.:

```
OP_IF <StandardTemplateHash(Tx B)> OP_CTV
OP_ELSE 2 <PK Bob> <PK Alice> 2 CHECKMULTISIG
OP_ENDIF
```
to

```
OP_IF <StandardTemplateHash(Tx B)> OP_CTV OP_DROP 1
OP_ELSE 2 
OP_ENDIF
<PK Bob> <PK Alice> 2 CHECKMULTISIG
```

This ensure's either Alice or Bob authorized the next step in the protocol if
the channel specification is fully public, or a salt can be added to the CTV
outputs (e.g., `<random data> OP_DROP`) if the amounts are known to the public
but not the entire scripts.

Even without either of these mitigations, the channel specification is secure
in the same threat model (t-delay broadcast to chain) as a regular lightning
channel.
