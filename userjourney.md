# User Journeys

In this chapter, you'll learn about the user-signed
contract method calls required to fulfill various 
actions you might like to do with the Computable protocol.

Let's start by noting that user-originated contract
method calls must be signed by the user’s private key,
which means you as the user are probably confirming
signing through a Metamask dialog. This is the
technique the first Computable dapps are using.

Second, contract method calls are asynchronous and may
take an indefinite amount of time since they only
complete when 1 (or really several) Ethereum blocks are mined. As you'll see in the [next chapter](../attacks/index.html), this can make a difference for security.

## Apply for a Listing

Here's the sequence of actions that happens upon listing application: 

- `Listing.list`: Apply for a listing. Voting happens
by other parties. Data is sent to Datatrust and which
calls setDataHash. Assume listing passes. Someone calls
Listing.resolveApplication and listing is ready to be
used.

After resolution, the applicant may claim the listing
reward (see Withdraw Support from a Market).


## Support a Market

Here's the sequence of actions a patron takes to
support a data market:

- `EtherToken.deposit`: Wrap ETH into an ERC20 compatible token so that contracts can manipulate it.

- `EtherToken.approve`: Call with deployed Reserve
  contract address to give that contract permission to
  change.

- `Reserve.support`: Convert `EtherToken` to
  `MarketToken` at price from bonding curve.


## Withdraw Support from a Market
Assume user has `MarketToken`. Note this flow applies
regardless of how `MarketToken` was acquired. User may
have supported the market with ETh or acquired by
creating a listing.

- `Listing.withdrawFromListing`: Call only if user has
  MarketTokens acquired through initial listing or
  listing access. Transfers MarketTokens to MarketToken
  contract so that it is available for withdraw. 

- `Reserve.withdraw`: Convert MarketToken to EtherToken
  at price from bonding curve.

- `EtherToken.withdraw`: Convert EtherToken to plain
  Eth.


## Vote on a Listing Candidate or Listing Challenge
Assume user has enough MarketToken to stake a vote, and
there is a listing to vote on. TODO: is this same
process for other votes like electing a DataTrust?

- `Voting.vote`: Supply ‘yeah’ or ‘nay’ vote as an
  argument. The user’s stake will be tied up during the
  voting period. 

- `Voting.unstake`: After the voting period is over,
  get the stake back.



## Challenge Listing
Assume challenger has enough `MarketToken` to stake the
challenge.

- `Listing.challenge`: Start the challenge.
  Stakeholders then vote. Someone must call
  Listing.resolveChallenge after the voting period.

- `Voting.unstake`: If the challenge succeeds, the
  challenger calls this method to get stake back. If the
  challenge failed, the listing owner calls this method
  to take the challenger’s stake.


## Buy Access to a Listing
- `EtherToken.deposit`: Wrap ETH into an ERC20
  compatible token so that contracts can manipulate it.

- `EtherToken.approve`: Call with deployed Datatrust
  contract address to give that contract permission to
  change.

- `Datatrust.requestDelivery`: “Deposit” funds to pay
  for access to listings at the market’s rate per byte.

User then tells Datatrust which listing to access.
Datatrust calls `Datatrust.listingAccessed`, which
deducts from the number of bytes user has paid for.
Datatrust delivers listing data to client and calls
`Datatrust.delivered` to collect its fee.


TODO, Other Governance Actions like reparameterize, elect DataTrust, etc

## Last Thoughts

Now that you have an understanding of the Computable
developer ecosystem, tools and flows, you should gain an
understanding of the security threats that face the
Computable ecosystem. Crypto always brings risks as
you'll learn in the next chapter.

<div style="text-align: right"> <a href="../../docs/attacks">Next Chapter</a> </div>
