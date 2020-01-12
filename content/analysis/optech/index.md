---
title: "Bitcoin Optech Newsletter #48"
date: 2019-10-10T17:31:05-07:00
author: <a href="https://bitcoinops.org">Bitcoin OpTech</a>
---


> *This article was orignally featured in Bitcoin Optech Newsletter #48, and has been
  excerpted, abbridged, and updated slightly for accuracy with the most recent iteration of
  the BIP for this page. The original OpTech content may be found
  [here](https://bitcoinops.org/en/newsletters/2019/05/29/). OpTech's content is
  offered under the [MIT License](https://opensource.org/licenses/MIT).*

The [proposed](https://github.com/JeremyRubin/bips/blob/op-secure-the-bag-master/bip-secure-the-bag.mediawiki) opcode `OP_CHECKTEMPLATEVERIFY` allows an address to commit to one or more branches that require the
transaction spending them to include a certain set of outputs, a
technique that contract protocol researchers call a *covenant*.
The primary described benefit of this proposed opcode is allowing a
small transaction to be confirmed now (when fees might be high) and have that transaction
trustlessly guarantee that a set of people will receive their actual
payments later when fees might be lower.  This can make it much more
economical for organizations that already implement techniques such as
[payment batching](https://github.com/bitcoinops/scaling-book/blob/master/x.payment_batching/payment_batching.md) to handle sudden fee spikes.

Before we look at the new opcode itself, let's take a moment to look at
how you might accomplish something similar using current Bitcoin
transaction features.

Alice wants to pay a set of ten people but transaction fees are
currently high so that she doesn't want to send ten separate
transactions or even use payment batching to send one transaction
that includes an output for each of the receivers.  Instead, she wants
to trustlessly commit to paying them in the future without having to pay
onchain fees for ten outputs now.  So Alice asks each of the receivers
for one of their public keys and creates an unsigned and unbroadcast *setup* transaction
that pays those keys using a 10-of-10 multisig script.  Then she creates
a child transaction that spends from the multisig output to the
10 outputs she originally wanted to create.  We'll call this child
transaction the *distribution* transaction.  She asks all the receivers
to sign this distribution transaction and she ensures each person
receives everyone else's signatures, then she signs and broadcasts the
setup transaction.

![A setup transaction paying a pre-signed distribution transaction](2019-05-tx-tree-1.png)

When the setup transaction receives a reasonable number of
confirmations, there's no way for Alice to take back her payment to the
10 receivers.  As long as each of the receivers has a copy of the
distribution transaction and all the others' signatures, there's also no
way for any receivers to cheat any other receiver out of a payment.  So
even though the distribution transaction that actually pays the
receivers hasn't been broadcast or confirmed, the payments are
secured by the confirmed setup transaction.  At any time, any
of the receivers who wants to spend their money can broadcast
the distribution transaction and wait for it to confirm.

This technique allows spenders and receivers to lock in a set of
payments during high fees and then only distribute the actual payments when
fees are lower.  According to Bitcoin Core fee estimates at the time of
writing, anyone patient enough to wait a week for a transaction to
confirm (like the distribution transaction above) can save significantly on fees.
Let's look at the example above in that context.  To make later
comparisons to Taproot more fair, we'll assume some form of key and
signature aggregation is being used, such as [MuSig][] or (in theory)
multiparty ECDSA (see [Newsletter #18][]).


|| Individual Payments | Batched Payment | Commit now, distribute later |
|--------|---------|----|-----|
| Immediate (high fee) transactions | 10x<abbr title="A transaction spending one P2WPKH input to two P2WPKH outputs">141 vbytes</abbr> | 1x<abbr title="A transaction spending one P2WPKH input to eleven P2WPKH outputs, or 9*(8+1+22) more bytes than the two-output P2WPKH transaction">420 vbytes</abbr> | 1x<abbr title="A transaction spending one P2WPKH input to two P2WPKH outputs">141 vbytes</abbr> |
| Cost at 0.00142112 BTC/KvB | 0.00204641 | 0.00059687 | 0.00020037 |
| Delayed (low fee) transactions | --- | --- | 1x<abbr title="A transaction spending one P2WPKH input to ten P2WPKH outputs, or 8*(8+1+22) more bytes than the two-output P2WPKH transaction">389 vbytes</abbr> |
| Cost at 0.00001014 BTC/KvB | --- | --- | 0.00000394 |
| Total vbytes | 1,410 | 420 | 530 |
| Total cost | 0.00204641 | 0.00059687 | 0.00020431 |
| **Savings compared to previous column** | --- | **71%** | **66%** |

We see that this type of trustlessly delayed payment can save 66% over
payment batching and 90% over sending separate payments.  Note that the
savings could be even larger during periods of greater fee
stratification or with more than ten receivers.

### OP_CHECKTEMPLATEVERIFY
The proposed soft fork would add a new opcode, `OP_CHECKTEMPLATEVERIFY` (abbreviated by
its author as `OP_STB`).  This opcode and a hash digest could be included in
tapleaf scripts, allowing it to be one of the conditions in a Taproot address.
When that address was spent, if OP_STB was executed, the spending transaction
would only be valid if the hash digest of its outputs matched the hash digest
read from the script by OP_STB.

Comparing this to our example above, Alice would again ask each of the
participants for a public key (such as a Taproot
address[^taproot-pubkeys]).  Similar to before, she'd create 10 outputs
which each paid one of the receivers---but she wouldn't need to form
this into a specific distribution transaction.  Instead, she'd just hash
the ten outputs together and use the resultant digest to create a
tapleaf script containing OP_STB.  That would be the only tapleaf in this
Taproot commitment.  Alice could also use the participants' public keys
to form the taproot internal key to allow them to cooperatively spend
the money without revealing the Taproot script path.

![A setup transaction paying a OP_STB output that expands into a distribution transaction](2019-05-tx-tree-2.png)

Alice would then give each of the receivers a copy of all ten outputs to
allow each of them to verify that Alice's setup transaction,
when suitably confirmed, guaranteed them payment.  When they later
wanted to spend that payment, any of them could then create a distribution
transaction containing the committed outputs.  Unlike the example from
the previous subsection, they don't need to pre-sign anything so they would never need to interact with each
other.  Even better, the information Alice needs to send them in order
to allow them to verify the setup transaction and ultimately spend their
money could be sent through existing asynchronous communication methods
such as email or a cloud drive.  That means the receivers wouldn't need
to be online at the time Alice created and sent her setup transaction.

This elimination of the need to interact is a particular highlight of
the proposal.  If we imagine the example above with Alice being an
exchange, the interactive form of the protocol would require her to keep
the ten participants online and connected to her service from the moment
each of them submitted their withdrawal request until the interaction
was done---and they'd all need to use wallets compatible with such a
child transaction signing protocol.  The non-interactive form with OP_STB
would only require them to submit a Bitcoin address and an email address
(or some other protocol address for delivery of the committed outputs).

### Feedback and activation

The proposal received over 30 replies on the Bitcoin-Dev mailing list as
of this writing.  The concerns raised included:

- **Not flexible enough:** Matt Corallo
 [says](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-May/016936.html),
 "we need to
  have a flexible solution that provides more features than just this, or we
  risk adding it only to go through all the effort again when people ask for a
  better solution."

- **Not generic enough:** Russell O'Connor suggests both OP_STB and
  `SIGHASH_ANYPREVOUT` (described in [last week's newsletter](https://bitcoinops.org/en/newsletters/2019/05/21/)) could be replaced using an `OP_CAT` opcode and an `OP_CHECKSIGFROMSTACK`
  opcode.  Both opcodes are currently implemented in
  ElementsProject.org sidechains such as [Liquid](blockstream.com/liquid)
  .  The `OP_CAT` opcode [catenates](https://en.wiktionary.org/wiki/catenate) two strings into one string
  and the `OP_CHECKSIGFROMSTACK` opcode compares a signature on the
  stack to other data on the stack rather than to the transaction that
  contains the signature.  Catenation allows a script to include various
  parts of a message that are combined with witness elements at spend
  time in order to form a complete message that can be verified using
  `OP_CHECKSIGFROMSTACK`.

    Because the message that gets verified can be a Bitcoin
    transaction---including a partial copy of the transaction the
    spender is attempting to send---these operations allow a script to
    evaluate transaction data without having to directly read the
    transaction being evaluated.  Compare this to OP_STB which looks at
    the hash of the outputs and anyprevout which looks at all the other
    signatures in the transaction.

    A potentially major downside of the cat/checksigfromstack approach
    is that it requires larger witnesses to hold the larger script and
    all of its witness elements.  O'Connor noted that he doesn't mind
    switching to more concise implementations (like OP_STB and
    anyprevout) once it's clear a significant number of users are making
    use of those functions via generic templates.

- **Not safe enough:** Johnson Lau pointed out that OP_STB allows
  signature replay similar to [BIP118](https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki) noinput, a perceived risk that
  [BIP AnyPrevout](https://github.com/ajtowns/bips/blob/bip-anyprevout/bip-anyprevout.mediawiki) takes pains to eliminate.

Rubin and others provided at least preliminary responses to each of
these concerns.  We expect discussion will be ongoing, so we'll report
back with any significant developments in future weeks.

The [proposed BIP for OP_STB][bip-coshv] suggests it could be activated
along with bip-taproot (if users desire it).  As bip-taproot is itself
still under discussion, we don't recommend anyone come to expect dual
activation.  Future discussion and implementation testing will reveal
whether each proposal is mature enough, desirable enough, and enough
supported by users to warrant being added to Bitcoin.

Overall, OP_STB appears to provide a simple (but clever) method for
allowing outputs to commit to where their funds can ultimately be sent.
In next week's newsletter, we'll look at some other ways OP_STB could be
used to improve efficiency, privacy, or both.



