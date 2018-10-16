The `Market` is the central contract that governs the behavior of a data
market. The current `Market` implementation has evolved from a `Registry`
implementation, but differs in a number of critical ways:

- The `Market` has an associated `MarketToken`. This `MarketToken` is created upon construction of the market. This token is minted and burned by various `Market` operations.
  - The [`MarketToken`](MarketToken.md) is a mintable and burnable ERC20 token.
- The `Market` holds a "reserve" to the Market. This reserve holds `NetworkToken` that is paid in by investors who want to take positions in market and will pay out to people who want to exit market.
  - `Market.invest(amount)` is a new method that allows an investor to enter the market.
  - `Market.divest()` allows any token holder to exit the market
- The market holds an "algorithmic price curve" that provides an automatic conversion rate from `NetworkToken` to `MarketToken`. The price curve is used by `Market.invest()` to determine the current conversion rate.
	- `Market.get_current_investment_price()` returns the current NetworkToken/MarketToken exchange rate from the price curve. This depends on the number of `NetworkToken` in the reserve.
- The `Market` supports a payment layer for queries against the underlying data market. Query payments must be performed via `Market` before backends will accept queries.
	- `T_util` units of `MarketToken` are minted for the owner of a listing when that listing is queried. The backend is responsible for reporting queried listings.

	- `Market.report_accessed(element_id)` can only be called by a backend for the market.
- Token holders in `Market` belong to one of two classes, data owner and investor.
	- `Market.convert_to_investor()` converts a data owner to an investor.
  - `Market.get_total_number_investor_tokens()` returns the total number of MarketTokens held by investors. This method will be used by Market.divest() and Market.get_current_investor_price()
  - `Market.set_access_cost(listing)`: Callable by the owner of a listing to set price for accessing the listing.
  - `Market.get_access_cost(listing)`: Getter to view cost.
