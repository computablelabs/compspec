`MarketToken` is a mintable and burnable ERC20 token. The `MarketToken` is tied to a particular `Market` and is created when the `Market` is created. Note the contrast with token curated registries, which don't hold a mechanism for minting and burning their associated token.

- Minting: Minting happens in one of three ways.
  - Minting happens when new listings are added to the market. These listings have to pass through the validation process and be whitelisted before minting occurs.
    - `listedReward` is the number of new `MarketTokens` that are created upon whitelisting.
  - Minting happens when an investor enters the market through calling `Market.invest()`. This rate is set by the algorithmic price curve.
  - Minting happens when the backend reports that a listing has been queried. This results in creation of `T_util` new tokens which are awarded to listing owner (which may be the market itself).
- Burning: Burning happens in one of two ways. 
  - If a listing is removed from the market, its associated tokens should be burned. This happens when the listing owner removes the listing or when a successful challenge forces removal of the listing.
  - If an investor class token holder divests from the market, their divested tokens are burned. The origin of the tokens being burned does not matter.
