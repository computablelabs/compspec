# Developer Libraries

Let's say that you're a developer. You've read through
the book so far, and you're excited to get started
working with the ecosystem. How do you get started?
Well the first place you want to get started is
probably interacting with some of the data markets that
are out there already. How can you do that?

Let's take a quick detour and talk through how
interactions with smart contract systems on Ethereum
work. Recall that Ethereum is a system that runs on
thousands of nodes, each of which verify and execute
every transaction that happens on the network. Talking
to a smart contract system involves talking with one of
these nodes, which will then propagate your transaction
out to the network at broad.

For this transaction to be actually entered into the
on-chain state, it needs to be "mined." That is, it has
to be processed by an Ethereum miner. We haven't
discussed Ethereum mining much, but for now, think of
it as a sort of "validation process" by which new
transactions are appended to the blockchain (a "block"
is basically a batch of transactions). Typically a
miner has to be incentivized to do this work (there's a
lot of competition for miner time), so transactions are
usually bundled with a gas fee which pays out to the
miner. This is worth pausing and thinking about. Every
interaction with the Ethereum ecosystem, and with the
Computable contracts, will cost you real money.

This creates an interesting set of dynamics. For
example, the set of transactions required to get a
listing candidate listed will take about $1 (more or
less) in gas fees. Nothing is free. This rule of thumb
should inform a lot of your thinking as you interact
with the ecosystem.

In particular, this means that you'll need to be set up
with some Ethereum basics. You'll need control of an
Ethereum address which has some funds to pay for gas
fees. The easiest way to do this is probably installing
an Ethereum wallet such as
[metamask](https://metamask.io/). You'll then need to
purchase some ETH from an exchange such as
[Coinbase](https://www.coinbase.com/) and transfer ETH
from your exchange wallet to your personal wallet. We
recommend starting with small amounts of money. It's
really easy to make mistakes and there are no
take-backsies on the blockchain.

Once you have a wallet set up, you'll still need a way
to programmatically interact with the Ethereum
ecosystem. The Ethereum community has built a number of
developer libraries to facilitate this access. The most
popular are probably
[web3.js](https://github.com/ethereum/web3.js/) and
[web3.py](https://github.com/ethereum/web3.py). These
libraries are very handy, but they're somewhat
low-level. They provide you the nuts and bolts you need
to construct Ethereum transactions from scratch. This
gives you a really useful substrate to do many things
with Ethereum, but it will take you some significant
effort with them before you can start interacting with
Computable contracts.

For this reason, we've built the [computable.py](https://github.com/computablelabs/computable.py) and [computable.js](https://github.com/computablelabs/computable.js) libraries. These libraries provide higher-order object oriented interfaces to the Computable contracts. For example, let's check out the `EtherToken` class in `computable.py`:

```

from computable.contracts.erc20 import ERC20

class EtherToken(ERC20):
    def at(self, w3, address):
        super().at(w3, address, 'ethertoken')

    def deposit(self, amount, opts=None):
        """
        @param amount An amount of ETH, in wei, sent as msg.value
        @param opts Transact Opts for this send type method
        """
        opts = self.assign_transact_opts({'gas': self.get_gas('deposit'), 'value': amount}, opts)
        return self.deployed.functions.deposit(), opts

    def withdraw(self, amount, opts=None):
        """
        @param amount An amount of ETH, in wei, to withdraw from this contract
        by its owner
        """
        opts = self.assign_transact_opts({'gas': self.get_gas('withdraw')}, opts)
        return self.deployed.functions.withdraw(amount), opts
```

We won't go into too much detail, but you can see how
the major functionality of `EtherToken` (depositing and
withdrawing funds) are exposed as methods. You can use
this and similar classes to easily write readable code
that interacts with the Computable contracts.

## Last Thoughts

You've seen a bit about how to interact with Computable
programmatically through the developer libraries. But
what are some of the flows you might want to know about
for getting things done? You'll learn more in the next
chapter

<div style="text-align: right"> <a href="../../docs/userjourney">Next Chapter</a> </div>
