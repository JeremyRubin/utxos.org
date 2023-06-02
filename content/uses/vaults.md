---
title: "Vaults"
date: 2019-10-18T10:45:51-07:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---

![](/images/uses/vaults.svg)
When greater security is required for cold storage solutions, there can be
default Tapscript paths that move funds from one target to another target.

For example, a cold wallet can be set up where one customer support desk can,
without further authorization, move a portion of the funds (using multiple
pre-set amounts) into a lukewarm wallet operated by an isolated support desk.
The support desk can then issue some funds to a hot wallet, and send the
remainder back to cold storage with a similar withdrawal mechanism in place.

This is all possible without OP_CHECKTEMPLATEVERIFY, but OP_CHECKTEMPLATEVERIFY
eliminates the need for coordination and online signers, as well as reducing the
ability for a support desk to improperly move funds.

Furthermore, all such designs can be combined with relative time locks to give
time for compliance and risk desks to intervene.

Check out the following for more information and code examples of Vaults:

### Implementations
- [simple client](https://github.com/jamesob/simple-ctv-vault)
- [python-vaults](https://github.com/kanzure/python-vaults)
- [Bitcoin Core
    Implementation](https://github.com/JeremyRubin/bitcoin/blob/checktemplateverify-feb1-workshop/src/wallet/rpcwallet.cpp#L1339)
- [Sapio Vault Tutorial](https://rubin.io/bitcoin/2022/03/22/sapio-studio-btc-dev-mtg-6/)
### General Information
- [Custody Protocols Using Bitcoin Vaults](https://arxiv.org/abs/2005.11776)
- [On-chain defense in
depth](https://github.com/kanzure/diyhpluswiki/blob/5bd7b874a02bbb9f22a76d30441a7c200ee5eb76/transcripts/2019-02-09-mcelrath-on-chain-defense-in-depth.mdwn)
- [Revault](https://github.com/kanzure/diyhpluswiki/blob/2a52f13ff4b3a19bca4c0239401192afe94c572a/transcripts/honey-badger-diaries/2020-04-24-kevin-loaec-antoine-poinsot-revault.mdwn)
- [Building Vaults on Bitcoin](https://rubin.io/bitcoin/2021/12/07/advent-10/)
- [Inheritence Schemes](https://rubin.io/bitcoin/2021/12/08/advent-11/)
