# Market Token

`MarketToken` is a mintable and burnable ERC20 token.
The `MarketToken` is tied to a particular `Market` and
is created when the `Market` is created. Note the
contrast with token curated registries, which don't
hold a mechanism for minting and burning their
associated token.

The `MarketToken` is denominated in "market wei".  As
with ETH, a "market wei" denominates 1/(10^18) of a
`MarketToken`. Using wei units throughout prevents
rounding error propagation and keeps contracts simple.

`MarketTokens` are dynamically minted and burned as the
market evolves. This flexibility is needed to
accurately track the evolving value of data in a data
market.  `MarketTokens` are minted in one of a few
scenarios explained below. In each case, the amount
minted is set by the `Parameterizer` which holds
`Market` parameters.

## Minting 
Minting happens when new listings are listed in the
market. These listings have to be approved by a
stakeholder vote. 

- Minting happens when a patron supports the market by
  making a payment into its reserve in `EtherToken` via
  `Reserve.support()`. The algorithmic price curve
  controls the exchange rate
  (`Reserve.getSupportPrice()`.) which governs the number
  of `MarketToken` consequently minted.
- Minting happens when a datatrust reports that a
  listing has been queried
  (`Datatrust.listingAccessed()`). The minted tokens are
  awarded to the listing owner.

## Burning

Burning happens in the scenarios explained below.

- If a token holder divests from the data market
  (`Reserve.withdraw()`), their divested tokens are
  burned. The origin of the tokens being burned does not
  matter.

[Next Chapter](../voting/index.html)
