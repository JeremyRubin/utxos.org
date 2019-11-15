---
title: "Batch Channels"
date: 2019-10-18T10:46:27-07:00
---
Using OP_SECURETHEBAG for Channel Factories is similar to the use for Congestion
Control, except the leaf nodes should be set up as a channels instead of plain
payments. The channel can be between the sender and recipient or a target of
recipient's choice. Using an OP_SECURETHEBAG, the recipient may even
give the sender an address which makes a set of channels unbeknownst to them.

These channels are already time insensitive for setup, as all punishments can be
relative timelocked to the actual instantiation. This enables set-up of channels
non-interactively in the case of a single funder.

This permits instant liquidity on the coins sent using this delayed method.

