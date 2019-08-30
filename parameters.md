# Market Parameters

As we've mentioned at various times in the previous
chapters, the data market is governed by a set of a
parameters dictated within `Parameterizer`. These
parameters govern the function of the market by setting
various critical settings.  These parameters can be
modified via a stakeholder vote.

In this chapter, we'll review the parameters that
govern the data market. Most of these should seem
similar, since you'll have run into them already
earlier in the book.

## stake

```
stake: wei_value
```
The stake (in `MarketToken` wei) needed to issue a
challenge to a listing. This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

## vote\_by

```
vote_by: timedelta
```
The time (in seconds) that a poll should remain open.
This controls the length of the voting window in which
council members can vote upon an `Market` listing,
challenge, or reparameterization. This parameter is of
type
[timedelta](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#time)

## plurality

```
plurality: uint256
```
The percent (whole number between 0 and 100) of the
vote needed by a candidate to pass in a poll. This
parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)

## price\_floor

```
price_floor: wei_value
```

The price floor for purchasing `MarketToken` via the
algorithmic price curve. This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)

## spread

```
spread: uint256
```
The spread which is rewarded to the market when
purchasing `MarketToken` via the algorithmic price
curve. This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)

## list\_reward

```
list_reward: wei_value
```
The number of new `MarketToken` wei that are minted
when a listing is listed. This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

## cost\_per\_byte

```
cost_per_byte: wei_value
```
The cost to purchase a byte of data from the market.
This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

## backend\_payment

```
backend_payment: uint256
```
The percentage of a delivery payment that's alloted to
the backend. Must be a percentage between 0 and 100.
This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)

## maker\_payment

```
maker_payment: uint256
```
The percentage of a delivery payment that's alloted to
the maker. Must be a percentage between 0 and 100.
This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)


## Reparameterization

All market parameters can be changed with a vote. The
process of changing data market parameters is referred
to as reparameterization. The same voting mechanism is
used for reparameterization as for other market
processes. That is, a reparameterization candidate is created, and put up for vote as described in the [voting chapter](../voting/index.html)

## Last Thoughts

We're finally done learning about all 7 contracts in
the Computable ecosystem! This is a big milestone. At
this point, you should have gained an understanding of
the on-chain economic dynamics that control a data
market. This means it's now time to start learning
about the off-chain parts of a data market. In
particular, let's pin down what a datatrust's off-chain
parts actually do. You'll learn more in the next
chapter.

<div style="text-align: right"> <a href="../../docs/capi">Next Chapter</a> </div>
