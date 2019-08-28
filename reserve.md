# Reserve

The `Reserve` is the "bank account" tied to each data
market. It holds the funds tied to that data market and
serves as incentive for makers to contribute the market.

Think of the reserve as holding earnings from the data
in the data market that belong to all the `MarketToken`
holders associated with the market. These earnings can
come from either data purchase payments or from patron
support for `MarketToken`. `MarketToken` holders are
allowed to withdraw earnings from the reserve by
burning their `MarketToken` holdings.

This section introduces the basics of the reserve for
each market. In addition, it introduces the "algorithmic price curve," the market making mechanism which allows stakeholders to buy or sell `MarketToken` at all time.

At present, the reserve is denominated in `EtherToken`.

## Support and Withdraw
```
@public
def support(offer: wei_value):
  """
  @notice Allow the purchase MarketToken with EtherToken priced according to the "buy-curve"
  @param offer An amount of Ether Token in Wei
  """
  price: wei_value = self.getSupportPrice()
  assert offer >= price # you cannot buy less than one billionth of a market token
  self.ether_token.transferFrom(msg.sender, self, offer)
  minted: uint256 = (offer / price) * 1000000000 # NOTE the ONE_GWEI multiplier here as well
  self.market_token.mint(minted) # TODO maybe implement `mintFor()`
  self.market_token.transfer(msg.sender, minted)
  log.Supported(msg.sender, offer, minted)
```

`Reserve.support()` consults the algorithmic price
curve to obtain the exchange rate.  Note that `offer`
is in units of `EtherToken` wei.  The returned value
will be in terms of `MarketToken` wei. `offer` will be
added to the data market reserve and the returned
`MarketToken` will be newly minted.

```
@public
def withdraw():
  """
  @notice Allows a supporter to exit the market. Burning any market token owned and
  withdrawing their share of the reserve.
  @dev Supporter, if owning a challenge, may want to wait until that is over (in case they win)
  """
  withdrawn: wei_value = self.getWithdrawalProceeds(msg.sender)
  assert withdrawn > 0
  # before any transfer, burn their market tokens...
  self.market_token.burnAll(msg.sender)
  self.ether_token.transfer(msg.sender, withdrawn)
  log.Withdrawn(msg.sender, withdrawn)
```

`Reserve.withdraw()` will burn all the `MarketTokens`
associated with this stakeholder and will withdraw
their share of the reserve (the percent of reserve
withdrawn equals the percent of `MarketToken` this
stakeholder owns).

More precisely, the fractional ownership this
stakeholder has is `num_tokens/total_num_tokens`. For
example, if `num_tokens=5` and `total_num_tokens=100`,
this would be 5% fractional ownership. Then
`num_tokens` market tokens are burned. Then the
fractional part of the reserve belonging to this
stakeholder is transferred to them. For example, in the
case above, 5% of the reserve would be transferred to
the stakeholder's address.


#### Algorithmic Price Curve
The price curve dictates the conversion rate between
`EtherToken` and `MarketToken` for new patrons. Patrons
purchase new `MarketToken` at the rate dictated by the
price-curve.

```
@public
@constant
def getSupportPrice() -> wei_value:
  """
  @notice Return the amount of Ether token (in wei) needed to purchase one billionth of a Market token
  """
  price_floor: wei_value = self.parameterizer.getPriceFloor()
  spread: uint256 = self.parameterizer.getSpread()
  reserve: wei_value = self.ether_token.balanceOf(self)
  total: wei_value = self.market_token.totalSupply()
  if total < 1000000000000000000: # that is, is total supply less than one token in wei
    return price_floor + ((spread * reserve * 1000000000) / (100 * 1000000000000000000))
  else:
    return price_floor + ((spread * reserve * 1000000000) / (100 * total)) # NOTE the multiplier ONE_GWEI
```

`Reserve.getSupportPrice()` reports the current
`EtherToken`/`MarketToken` conversion rate.
Mathematically, the idea is that the `support()` price
will be slighly higher than the `withdraw()` price.
Precisely, if `spread` is 110, then there will roughly
be a 10% overhead. This ratio isn't quite precise due
to the need to handle edge conditions on the price
curve.

[Next Chapter](../datatrust/index.html)
