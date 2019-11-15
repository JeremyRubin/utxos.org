---
title: "Vaults"
date: 2019-10-18T10:45:51-07:00
---

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

