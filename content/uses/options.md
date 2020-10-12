---
title: "Decentralized options"
date: 2020-10-11T10:45:58-07:00
author: <a href="https://github.com/corollari">corollari</a>
---
![](/images/uses/options.png)

Trading of bitcoin derivatives is a huge market nowadays. However, all of it occurs on centralized exchanges with large legal & technical risks that require trust in their operators, thus betraying the philosophy of Bitcoin. The protocol presented here enables the creation and redemption of trustless options on the Bitcoin network.

## Main protocol
Option creation will follow this protocol:
1. Use an atomic swap to sync these two operations:
  - User pays fee to underwriter
  - Underwriter sends tokenized USD [1] to an address that is encumbered by the script:
      - After X+K blocks:
        - Send to underwriter
      - After X blocks:
        - If spending tx sends Y BTC to underwriter, send to user
2. When the option reaches maturity the user can exercise it by sending a transaction that:
  - Sends BTC to underwriter
  - Spends the UTXO created on step 1 and sends it to the user's wallet
3. If the user doesn't exercise it on time the underwriter can just spend it to itself

## Reduction of capital requirement
As it is, this protocol requires locking 100% of the collateral for as long as the option stays in effect, but it would also be possible to reduce the capital requirement quite substantially by changing the protocol to make the underwriter lock only 30-50% initially and, at option expiration, make it provide the rest of the payment in case the user decides to exercise the option. More concretely, this new protocol would look like:
1. Atomic Swap:
  - User pays fee to underwriter
  - Underwriter sends 50% of the option's collateral in BTC to an address encumbered by the script:
      - After X+K blocks:
        - Send to underwriter
      - After X blocks:
        - If tx has an UTXO that spends 100% of the option's value in BTC, consolidate all UTXOs (150% value) and send it to another address encumbered by the following script:
          - If tx sends the required USD to the user, send the BTC UTXO to the underwriter
          - After K blocks:
            - Send all the value to the user, the extra 50% acts as a punishment for the underwriter
2. When the option reaches maturity the user can exercise it by sending a transaction that adds BTC to the money locked by the underwriter previously and moves it into another state
3. After the funds have moved to the "exercised" state, the underwriter will have to provide the promised USD to the user in exchange for BTC, as not doing so will result in them losing the initial 50% deposit

As you have probably already noticed, the downside of this system is that it would require an external oracle along with a re-balancing mechanism to prevent situations where losing the initial deposit is the rational choice for underwriters. Furthemore, it would introduce the assumption that price must not change by over X% within a pre-defined timespan.

## Notes
[1] The USDT available on OmniLayer wouldn't work for this, since OmniLayer only uses BTC transactions for serialization and propagation (the payload is encoded in the txs in a way that is invisible to BTC nodes) so it won't be possible to apply BTC's script to these, instead, some sort of USD colored coin needs to be used.
