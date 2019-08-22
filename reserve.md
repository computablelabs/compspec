# Reserve

This section introduces the basics of the reserve for
each market.

		- [Market Reserve](#market-reserve) [v0.2]: The reserve is the "bank account" associated with a given `Market`. 
		- [Algorithmic Price Curve](#algorithmic-price-curve) [v0.2]: Controls the price at which new investors may invest in market. Investor funds are deposited in reserve and new market token is minted accordingly.

#### Reserve
The `Market` holds with it an associated "reserve." Think of the
reserve as holding earnings from the data in the `Market` that belong
to all the `MarketToken` holders associated with the market. These
earnings can come from either query payments or from investor
purchases of `MarketToken`.  Investor class `MarketToken` holders are
allowed to withdraw earnings from the reserve by burning their
`MarketToken` holdings.

At present, the reserve is denominated in [`NetworkToken`](#network-token).

#### Support and Withdraw
```
function invest(uint offered) external returns (uint)
```
`Market.invest()` consults the [algorithmic price
curve](#algorithmic-price-curve) to obtain the exchange rate This
method can only be called from an address which is not already a
listing owner. If the call succeeds, it will add a new investor class
member (if not already added).  Note that `offered` is in units of
`NetworkToken` wei.  The returned value will be in terms of
`MarketToken` wei. `offered` will be added to the `Market` reserve and
the returned `MarketToken` will be newly minted.

```
function divest() external returns (uint)
```
`Market.divest()` will check if the caller is investor class. If so,
it will burn all the `MarketTokens` associated with this investor and
will withdraw the investor's share of the reserve (the percent of
reserve withdrawn equals the percent of investor class `MarketToken`
this investor owns).

More precisely, the fractional ownership this investor has is
`num_tokens/total_num_investor_tokens`. For example, if `num_tokens=5`
and `total_num_investor_tokens=100`, this would be 5% fractional
ownership. Then `num_tokens` market tokens are burned. Then the
fractional part of the reserve belonging to this investor is
transferred to the investor. For example, in the case above, 5% of the
reserve would be transferred to the investor's address.


#### Algorithmic Price Curve
The price curve dictates the conversion rate between `NetworkToken`
and `MarketToken` for new investors. Investors purchase new
`MarketToken` at the rate dictated by the price-curve.


![The Algorithmic Price Curve](Algorithmic_Price_Curve.png)


```
function get_current_investment_price() pure returns (uint)
```
`Market.get_current_investment_price()` reports the current
`NetworkToken`/`MarketToken` conversion rate. Mathematically, the
first version will be a linear function. That is,
`Market.get_current_investment_price() = base_conversion_rate +
conversion_slope * Market.get_reserve_size()` where
`base_conversion_rate` and `conversion_slope` are parameters defined
by the market creator in the `Parameterizer`.

```
function get_reserve_size() view returns (uint)
```
`Market.get_reserve_size()` returns the size of current market reserve in `NetworkToken` wei


