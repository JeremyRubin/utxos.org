---
title: "Trustless Coordination-Free Mining Pools"
date: 2019-10-18T10:45:58-07:00
author: <a href="https://twitter.com/JeremyRubin">Jeremy Rubin</a>
---
![](/images/uses/bagpool.svg)

_for more detail on decentralized mining pools in a more recent article [read here](https://rubin.io/bitcoin/2021/12/12/advent-15/)_


In a typical mining pool, miners works together to create a shared block reward. The shared reward
is distributed among participants based on difficulty shares, that is, partial work proofs that
"prove" a miner was working on an appropriate block.

An appropriate block pays the reward to the pool operator, who then distributes (less a fee) the
rewards to the other miners.

Using `OP_CHECKTEMPLATEVERIFY`, miners can instead look at a set of *qualified* historical blocks
deterministically (e.g., the last N, or all blocks not older than some time T, or blocks between T1
and T2) and share their block reward with the miner behind those blocks. `OP_CHECKTEMPLATEVERIFY` is
an enabling technology here because it is cheap to issue remittences to all of these miners. A block
would be considered qualified if it also followed the same pooling rule.

To participate in such a pool, a miner has to have enough hashpower to mine at least one block to
enter the pool. Then, the system can reduce the variance that a miner is exposed to, as demonstrated
in the picture. Trusted mining pools can still exist to get miners up to a reasonable level of
hashrate to participate in the trustless pools effectively, but these pools would be smaller and
bear less risk.

This type of mining pool has no trusted operator and requires no coordination. This eliminates
various types of mining pool attacks, such as block witholding.


The below code shows a simple simulation of experienced variances among a set of miners with a fixed
sharing policy versus a set of miners without any sharing.


```python
import numpy as np
from matplotlib import pyplot as plt
# TODO: Handle halvings?
BLOCK_REWARD = 12.5
STANDARD_SHARE = 0.75

AVG_INTERVAL = 10.0*60.0
class Block:
    def __init__(self, miner_id, height, shares, percent_shares, reward, lookback, time):
        self.miner_id = miner_id
        self.height = height
        self.shares = shares
        # Compute our personal take of the revenue
        self.revenue = (1-percent_shares)*reward if shares else reward
        self.percent_shares = percent_shares
        self.shares = []
        self.time = time
        if percent_shares != 0 and shares:
            total_avail = percent_shares * reward
            total_percent = sum(block.percent_shares for block in lookback)
            # If the prior blocks were in the pool
            if total_percent != 0:
                # Pay past miners something for their help
                # based on the percent they shared of the total percent shared to us.
                # TODO: Maybe EWMA is better than straight average?
                self.shares = [(block.miner_id, block.percent_shares/total_percent * total_avail)
                                for block in lookback]
            else:
                # Before the pool exists...
                # We still give them something (to solve the bootstrap problem...)
                # This could be done by issuing forward-looking block height locked payments
                # to future miners or by one-time coordination
                self.shares = [(block.miner_id, 1.0/len(lookback) * total_avail) 
                                for block in lookback]


def sim(n_miners = 100, p_shares = 0.5,
        n_blocks = 10000, n_blocks_lookback=144,
        share_amt = 143.0/144, power=1.5):
    # Make an array of miners IDs
    miners = np.array(range(n_miners))

    # Distribute hashrate on some power law and normalize so probabilities sum to 1
    miner_hashrate = np.random.power(power, n_miners)
    miner_hashrate /= np.sum(miner_hashrate)

    # Flip a biased coin to determine if a miner is in the pool or not
    miner_shares = np.random.random(n_miners) <= p_shares

    # A list of all blocks (a... block... chain)
    blocks = []
    # A queue for just the last n_blocks_lookback sharing blocks
    sharing_blocks = []
    block_time = 0
    # Initialize the sharing blocks queue
    while len(sharing_blocks) == 0 or 
        sharing_blocks[-1].time-sharing_blocks[0].time >= n_blocks_lookback*AVG_INTERVAL:

        miner = np.random.choice(miners, p = miner_hashrate)
        block_time += np.random.poisson(AVG_INTERVAL)
        # we share 0 percent here because we don't know how many there are...
        blocks.append(Block(miner, len(blocks), miner_shares[miner],
                      0.0, BLOCK_REWARD, sharing_blocks, block_time))
        if miner_shares[miner]:
            sharing_blocks.append(blocks[-1])

    # Now mine n_blocks
    for x in range(n_blocks):
        miner = np.random.choice(miners, p = miner_hashrate)
        block_time += np.random.poisson(AVG_INTERVAL)
        blocks.append(Block(miner, len(blocks), miner_shares[miner],
                            share_amt, BLOCK_REWARD, sharing_blocks, block_time))
        if miner_shares[miner]:
            sharing_blocks.append(blocks[-1])
            if sharing_blocks[-1].time-sharing_blocks[0].time >= 
                n_blocks_lookback*AVG_INTERVAL:
                sharing_blocks.pop(0)
    tabulate_and_show(n_miners, miner_shares, miner_hashrate, blocks)
# Compute the revenues
def tabulate_and_show(n_miners, miner_shares, miner_hashrate, blocks):
    # have a column for each miner with space for an entry per block
    revenues = np.zeros((n_miners, len(blocks)))
    for block in blocks:
        h = block.height
        # give the miner their revenue
        revenues[block.miner_id, h] += block.revenue
        # if the block shares, assign the shares to each miner
        for (to, amt) in block.shares:
            revenues[to, h] += amt
    # Compute the column variance on the revenues
    norm = np.outer(miner_hashrate, BLOCK_REWARD* np.ones((len(blocks)))).clip(1)
    var = np.var(revenues/norm , axis=1)
    # Show the averages for just the sharing miners
    # and then just the non sharing miners
    # TODO: better measure of difference than average?
    #       Perhaps weighted average by hashrate?
    print("Avg var, sharing miners %f"% (np.sqrt(np.average(var*miner_shares))/70.0))
    print("Avg var, solo miners %f"%(np.sqrt(np.average(var*(miner_shares ^ 1)))/70.0))
    # cumsum the instantaneous revenues to get something nice for charting
    chart = np.cumsum(revenues, axis=1)

    fig, host = plt.subplots()
    host.set_title("Decentralized Pool Visualization ")
    host.set_xlabel("Blocks")
    host.set_ylabel("Adjusted Normalized Revenue:\n(R / max(E[R], 1))")
    host.set_ylim(0.8,1.2)
    for (i,col) in enumerate(chart):
        # Show pooled miners as dotted, not pooled as unbroken
        earnings = (np.array(range(len(blocks)))*miner_hashrate[i]*BLOCK_REWARD).clip(1)
        if miner_shares[i]:
            host.plot(np.array(range(10000)), ((col/earnings)- 0.1)[-10001:-1], 'k')
        else:
            host.plot(range(10000), ((col/earnings)+0.1)[-10001:-1], 'r')
    plt.show()


plt.rcParams["font.size"] = "30"
sim(100, 0.5, 100000, 144*3, 1.0, 2)
```
