# Market Token

`MarketToken` is a mintable and burnable ERC-20 token
(we'll say more on what those terms mean soon).  The
`MarketToken` is tied to a particular data market and
is created when the data market is created. What is the
purpose of `MarketToken`? Well simply put, the
`MarketToken` corresponds to "shares" in a dataset. In
everyday life, you've probably bought share in
companies. These shares give you some rights. You can
vote on some company decisions, and have rights to some
portion of the company's wealth (albeit in limited
form). The `MarketToken` is similar. It gives you
partial ownership of the dataset that is hosted in the
data market along with some governance rights. We'll
explain the function of the `MarketToken` in greater
detail in this and subsequent chapters.

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

## Comparing with Token Curated Registries

Those of you who have some experience in crypto might
wonder if `MarketToken` is something like a [token
curated registry](https://medium.com/@tokencuratedregistry/a-simple-overview-of-token-curated-registries-84e2b7b19a06). There are some similarities here, since a data market also manages an on-chain "registry."

While our original
[research](https://arxiv.org/abs/1806.00139) on data
market design used token curated registries as a design
guide, the current design is quite different. In
particular, note the contrast of `MarketToken` with
token curated registries tokens, which don't have a
mechanism for minting and burning their associated
token.

[Next Chapter](../voting/index.html)
