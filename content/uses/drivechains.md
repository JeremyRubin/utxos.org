---
title: "Drivechains"
date: 2020-10-11T10:45:58-07:00
author: <a href="https://github.com/corollari">corollari</a>
---
![](/images/uses/drivechains.png)

Drivechain's peg-out mechanism could be implemented at the transaction level through scripts that force the funds being withdrawn from the sidechain to go through a chain of N transactions, each one of these being in a different block. More specifically, this would be implemented by creating N scripts that would contain an integer constant and would only be spendable to another copy of the same script with the constant increased or decreased by one, with the exception of the script where the constant reaches the value N, in which case the UTXO would be spendable only to a specific address provided in the beginning of the process. Furthermore, these scripts would prevent the possibility of several of them being chained in the same block through the use of `OP_CSV`.

Thus the protocol would look like this:
1. Users that want to peg money into the drivechain would send their funds to address_X. address_X's script would only allow the spending of coins to a `ScriptPubKey` that would follow the template `DCScript(our_address, 1)`, where `our_address` is a bitcoin address
2. Users that then want to retrieve money from the sidechain would start that process by creating a UTXO that would transfer some of the funds in address_X to an address created by computing `DCScript(our_address, 1)` where `our_address` is an address that would be spendable on a payout to the group of users that created this request (enforced through OP_CTV)
3. Users would need to start sending transactions spending UTXOs locked in `DCScript(our_address, i)` to `DCScript(our_address, i+1)` until reaching `DCScript(our_address, N)` at which point they could finally send the funds to `our_address`. The rationale behind drivechain also applies here, compelling miners to monitor these withdrawals and make sure that they are legal.
4. The funds could also be transferred from `DCScript(our_address, i)` to `DCScript(our_address, i-1)` and from `DCScript(our_address, 1)` back to `adress_X`, reversing the process. Users along with miners may want to do that in order to prevent an illegal withdrawal.

Pseudocode of `DCScript(address_arg, i)`:
```
OP_CSV 1 //Prevent several scripts from being chained in the same block
IF i <= 0 && output.address == address_X:
  SUCCEED
IF i >= N && output.address == address_arg:
  SUCCEED
IF output.address == DCScript(address_arg, i+1) || output.address == DCScript(address_arg, i-1):
  SUCCEED
FAIL
```

Note that, due to the restrictions inherent to OP_CTV, the set of possible addresses available to use as `our_address` must be finite and picked in the initial setup phase. Furthermore, as each possible state requires some computation, the length of the paths that a transaction can take to redemption is limited in depth and the size of any withdrawal must be within a set.
