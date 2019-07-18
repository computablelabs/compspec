All the contracts for the protocol currently deployed are in the
computable repo. We are currently only deploying to a private test
network, but as we deploy them to public test-nets and, eventually,
main-net, we will update the README of this repo with their addresses
and any other related information.

For the more dev-minded amongst you, our contracts are actively
developed here. We’ve built a number of developer utilities in there,
including a custom testing framework that integrates closely with
Geth’s tooling to provide a believable simulator engine. If you’ve
ever wondered how to compile, deploy and test your Vyper smart
contracts with an alternative to popular Javascript frameworks, this
is how we do it.

The “data market” itself is split across a series of 7 core contracts.
You might ask why we didn’t just create one monolith contract that
implements the data market. Well, monoliths don’t scale. As complexity
grows it would be harder and harder to devise abstractions and
permissions controls. More importantly however, and this you may not
know, there is a ~24 kb limit on the size of a smart contract on the
EVM. There are more reasons, but most are simply based around the
proper design of good software. We can save those discussions however
for elsewhere in the dev forums. Moving on: The contracts are
interlinked in the following fashion

![Contract Diagram](contracts.png)

In this diagram, arrows correspond to dependencies. An outgoing arrow indicates that this contract is dependent on the contract on the other end. The contracts are numbered such that contract i is not dependent on contract j for j > i. We’ll discuss the contracts in order:

- EtherToken: This contract is a form of wrapped Ether. It wraps ETH into an ERC20 interface that makes it easy to work with. The first step for working with the data market protocol is to transfer some ETH into EtherToken.
- MarketToken: This contract is an ERC20 that tracks ownership in the current data market.
- Voting: This contract governs voting in the data market. Recall that votes control a number of critical operations in the protocol, such as the decision of whether to admit a new listing to the market, and whether to change the core parameters that govern the market.
- Parameterizer: This contract governs the set of parameters which control the market’s behaviors.
- Reserve: This contract governs the data market’s reserve. In particular, this contract exposes the “algorithmic price curve” which governs how patrons can support and withdraw from the market.
- Datatrust: This contract governs the interaction of the datatrust with the core market. In particular, the “delivery flow” which controls how data is purchased is handled by this contract.
- Listing: This contract governs the addition of new chunks of data (“listings”) into the data market.

You might notice that some of the contracts have other contract names
specified in small red font underneath. These contracts have
“privilege” in the current contract. Contracts with privilege have the
ability to call certain special methods in the current contract. For
example, Listing and Reserve have privilege with the MarketToken
contract. Precisely, this means that these contracts have the ability
to mint and burn MarketToken. This privilege structure is needed to
implement the algorithmic buy/sell curve for patrons. As you’re
reading through the contracts, you might find it useful to keep this
privilege structure in your head so you can understand how the set of
contracts interlocks with one another. It is also worth noting that
the privilege mechanism serves in a security role as well, preventing
random accounts from having the ability to perform certain actions
(like mint tokens).

One of the differentiating factors of our implementation is that we’ve
chosen to use Vyper for our contract implementation. There were a
number of factors that drove this choice. For one, the fact that Vyper
defaults to safe math made our implementation of the algorithmic price
curve dramatically simpler. Although Vyper is still beta software,
we’ve found that it’s well written, well maintained and well suited to
our efforts.