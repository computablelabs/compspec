# Market Parameters

The data market is governed by a set of a parameters
dictated within `Parameterizer`. These parameters
govern the function of the market by setting various
critical settings.  These parameters can be modified
via a stakeholder vote. Let's review the parameters
controlled in `Parameterizer`:


```
stake: wei_value
```
The stake (in `MarketToken` wei) needed to issue a
challenge to a listing. This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

```
vote_by: timedelta
```
The time (in seconds) that a poll should remain open.
This controls the length of the voting window in which
council members can vote upon an `Market` listing,
challenge, or reparameterization. This parameter is of
type
[timedelta](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#time)

```
plurality: uint256
```
The percent (whole number between 0 and 100) of the
vote needed by a candidate to pass in a poll. This
parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)


```
price_floor: wei_value
```

The price floor for purchasing `MarketToken` via the
algorithmic price curve. This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)



```
spread: uint256
```
The spread which is rewarded to the market when
purchasing `MarketToken` via the algorithmic price
curve. This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)


```
list_reward: wei_value
```
The number of new `MarketToken` wei that are minted
when a listing is listed. This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

```
cost_per_byte: wei_value
```
The cost to purchase a byte of data from the market.
This parameter is of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei)

```
backend_payment: uint256
```
The percentage of a delivery payment that's alloted to
the backend. Must be a percentage between 0 and 100.
This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)

```
maker_payment: uint256
```
The percentage of a delivery payment that's alloted to
the maker. Must be a percentage between 0 and 100.
This parameter is of type
[uint256](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#unsigned-integer-256-bit)


#### Reparameterization

All market parameters can be changed with a vote. The
process of changing data market parameters is referred
to as reparameterization. The same voting mechanism is
used for reparameterization as for other market
processes.
