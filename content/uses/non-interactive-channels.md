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
