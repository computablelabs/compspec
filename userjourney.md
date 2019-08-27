The scope of this page is to list the user-signed
contract method calls required to fulfill various user
actions.

First, user-originated contract method calls must be
signed by the user’s private key, which means the user
probably confirms signing through a Metamask dialog.

Second, contract method calls are asynchronous and may take an indefinite amount of time since they only complete when 1 (or really several) blocks are mined. 

Actions and contract method calls made by Datatrust are
mentioned here, but the exact sequence of client,
protocol, and Datatrust actions is beyond scope of this
doc.


## Apply for a Listing
- `Listing.list`: Apply for a listing. Voting happens
by other parties. Data is sent to Datatrust and which
calls setDataHash. Assume listing passes. Someone calls
Listing.resolveApplication and listing is ready to be
used.

After resolution, the applicant may claim the listing reward (see Withdraw Support from a Market).


## Support a Market
- `EtherToken.deposit`: Wrap ETH into an ERC20 compatible token so that contracts can manipulate it.

- `EtherToken.approve`: Call with deployed Reserve
  contract address to give that contract permission to
  change.

- `Reserve.support`: Convert `EtherToken` to
  `MarketToken` at price from bonding curve.


## Withdraw Support from a Market
Assume user has `MarketToken`. Note this flow applies
regardless of how `MarketToken` was acquired. User may
have supported the market with Eth or acquired by
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
Assume user has enough MarketToken to stake a vote, and there is a listing to vote on. TODO: is this same process for other votes like electing a DataTrust?

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
