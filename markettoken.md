# Market Token

`MarketToken` is a mintable and burnable ERC20 token. The
`MarketToken` is tied to a particular `Market` and is created when the
`Market` is created. Note the contrast with token curated registries,
which don't hold a mechanism for minting and burning their associated
token.

As with the `NetworkToken`, the `MarketToken` is denominated in
"market wei".  As with ETH, a "market wei" denominates 1/10^18 of a
`MarketToken`. Using wei units throughout prevents rounding error
propagation and keeps contracts simple.
