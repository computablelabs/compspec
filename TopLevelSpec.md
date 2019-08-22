# The Computable Protocol



## Top Level Specification

  - [On-chain smart contracts](#on-chain-components):
		- [Market Parameters](#market-parameters): The `Market` is governed by a set of a parameters dictated within the `Parameterizer`.
        - [Reparameterization](#reparameterization): The parameters that govern the `Market` can be modified with a council vote.
	- [Datatrust](#datatrust-specification): A `Datatrust` is responsible for securely storing data off-chain and allowing authorized users to query this data. Note that a `Datatrust` may serve multiple markets. The `Datatrust` is an off-chain system that responds to the API specified in this document, and which understands how to interact with the on-chain Computable contracts.
      - [Authentication](#authentication): `Backends` should allow users to authenticate with them. 
      - [Storage](#storage): `Backends` must be able to persist off-chain data securely.
      - [Encryption at Rest](#encryption-at-rest): All stored data must be encrypted.
      - [Computational Workloads](#computational-workloads) [v0.3]: A `Backend` must be able to run computational workloads against its data. 
      - [REST API](#rest-api) [v0.3]: The `Backend`must respond to a defined set of REST API commands to perform actions such as authentication, data addition and removal, and query handling 
- [Forward Looking Research](#forward-looking-research): Features in this section are currently being researched with the goal of eventual inclusion into the core Computable protocol. However, these features are not yet formally on the roadmap for any given Computable version release.
  - [Fine Grained Data Utilization](#fine-grained-data-utilization): How can we track data utilization in a fine grained fashion.
  - [Query Rake](#query-rake): What fraction of the payment goes to each stake holder?
  - [Epsilon Privacy Curve](#epsilon-privacy-curve): A curve that prices queries by the amount of privacy loss they cost to the data market owner.
  - [Untrusted Backend](#untrusted-backend): A `Backend` system which is not trusted by the owners of the data market.
- [Case Studies](#case-studies) We consider a few case studies of interesting data markets that can be constructed with the Computable protocol in this section.
  - [Censorship Resistant Data Market](#censorship-resistant-data-markets) The Computable protocol allows for the construction of data markets that are resistant to censorship efforts.



### On Chain Components

The on-chain components of the protocol control
economics and access control.  If a user wants to gain
access to a particular dataset (in a particular data
market), or if a user wants to invest in a particular
data market, they have to seek on-chain authorization.
If a user wants to pay for queries, this is also done
off-chain. The advantage of this structure is that
payments and authorization can be handled securely by
secure on-chain contracts.

#### Market 

- The `Market` has an associated `MarketToken`. This `MarketToken` is created upon construction of the market. This token is minted and burned by various `Market` operations.The [`MarketToken`](#market-token) is itself a mintable and burnable ERC20 token.

- The `Market` has an "algorithmic price curve" that provides an automatic conversion rate from `NetworkToken` to `MarketToken`. The price curve is used by `Market.invest()` to determine the current conversion rate. The current conversion rate depends on the current size of the reserve.
- `MarketToken` holders in `Market` belong to one of two classes, data owner and investor. Only data owners can own listings in the market, and only investors have the right to withdraw from the reserve. A data owner can convert into investor class by giving up ownership of their listings. 

#### Minting and Burning Mechanics
`MarketTokens` are dynamically minted and burned as the `Market` evolves. This flexibility is needed to accurately track the evolving value of data in a data market.

`MarketTokens` are minted in one of a few scenarios explained below. In each case, the amount minted is set by the `Parameterizer` which holds `Market` parameters.
- Minting happens when new listings are listed in the market. These listings have to be approved by a council vote. 
- Minting happens when an investor invests in the market by making a payment into its reserve in `NetworkToken`. The algorithmic price curve controls the exchange rate which governs the number of `Markettoken` consequently minted.
- Minting happens when a `Backend` reports that a listing has been queried. The minted tokens are awarded to the listing owner.
 
Burning happens in the scenarios explained below.
- If a listing is removed from the `Market`, its associated tokens are burned. This happens when the listing owner removes the listing or when a successful challenge forces removal of the listing.
- If an investor class token holder divests from the `Market`, their divested tokens are burned. The origin of the tokens being burned does not matter.
 
#### Voting
Major decisions in the `Market` are made by token holder vote.  These
decisions include which new listings should be added to the `Market`,
which challenged listings should be removed, and what changes should
be made to the `Market` parameters.

The votes here are *not* stake-weighted. All council members have
precisely one vote. So a council member with `5*T_council` and another
council member `1.1*T_council` `MarketTokens` have the same voting
power. In addition, all council votes at present are cast publicly
with no lock-commit-reveal scheme.  This allows for the implementation
of a simple voting mechanism with smaller attack surface.


#### Listings 
A market holds a set of `Listings`. Each listing corresponds to an element of the
`Market` which is held off-chain in some (possibly multiple) `Backend` systems.
Newcomers to the market can call `Market.apply()` to apply to have their
listing added to the market. A listing consists of an off-chain datapoint (or
datapoints) and an on-chain listing structure. (We haven't defined "datapoint"
here yet.) We reproduce the fields of the on-chain listing structure below.

```
struct Listing {
  bool listed; // a 'listing' if true
  address owner; // owns the listing
  uint supply; // Number of tokens in the listing (both deposited and minted).
  uint challenge; // corresponts to a poll id in Voting if present
  bytes32 dataHash; // Hash of the off-chain data-point this listing corresponds to
  uint rewards; // Number of Market tokens that have been minted for this listing.
}
```

Let's take a minute to walk through the fields of this struct to
explain how the `Listing` works. The `Listing` is an on-chain record
of a chunk of off-chain data. The `dataHash` is the hash of the set of
off-chain data that this listing corresponds to. For our purposes,
this off-chain data is simply an arbitrary blob (a bytestring of
arbitrary length) that is hashed down to a single `bytes32` value.

The `listed` boolean field specifies whether this listing is
officially listed or not in this given market. The `owner` field is
the market participant who owns this listing. If this owner has
converted to investor class, ownership of the listing will be
transferred to the market itself and the `address` in this field will
be the market address.

The `supply` field is the number of `MarketToken` that the listing
proposer is willing to stake to see this listing listed in the
`Market`. This must exceed the `minDeposit` that is demanded by the
`Parameterizer` tied to this market.  The purpose of this stake is to
reward challengers who remove useless listings from a given market.

The `challenge` field tracks if there's an active challenge to this
`Listing` at present. `rewards` tracks how many new `MarketToken` have
been minted for this `Listing`. Note that this field is only nonzero
for `Listings` which have successfully been listed.

Let's pause here and say a few words about the has function used to
generate `dataHash`. It's important that this hash function be a
cryptographic hash function which is collision resistant. This means
that given `dataHash`, it isn't feasible to spoof a fake datapoint
that has the same hash. This means that `dataHash` can be treated as a
unique identifier of the datapoint.

In particular, `dataHash` must be computed with KECCAK-256. This is
the same hash function that solidity uses on-chain.

![Maker Flow](Maker_Flow.png)

#### What is a datapoint?
We haven't clearly specified what a "datapoint" is in the preceding
material.  Part of the challenge is that a "datapoint" will mean
different things for different markets. A record in an off-chain SQL
database is very different from an image file for a deep learning
`Backend`. For this reason, we say that the "datapoint" tied to a
listing is simply an arbitrary bytestring. This bytestring may
correspond to multiple "logical datapoints".  For example, the
bytestring may correspond to 10 SQL rows or to 50 images. This
batching might be crucial for efficiency, since the transaction rate
of Ethereum is not yet sufficient to do bulk uploads of datasets
otherwise.

#### Applying
Applying is the process by which a new listing is added to a data
market. To apply, a market participant computes the hash of their
off-chain data and proposes the addition of their data to the market
by invoking `Market.apply()`:

```
function apply(bytes32 listingHash, uint amount, string data) external
```

All applications trigger a vote on the new listing by appropriate
market stakeholders (either all token-holders or the market council).
If a listing vote is cleared, it is said to be listed. Note that
application is a *minting* event whereby new `MarketTokens` are
created. More detail on this can be found in the section on minting.

#### Challenging
Challenging is the process by which a listing in a data market can be
challenged and potentially removed. A challenge triggers a vote. If
the challenge succeeds, the challenged listing is de-listed from the
data market.  If the challenge fails, the challenging party is
penalized with a loss of stake (note that posting a challenge requires
placing `MarketToken` at stake).

Note that unlike a token curated registry, the council receives no
reward for voting upon a challenge. Only the victor of the challenge
receives a financial reward which comes directly from the loser of of
the challenge.

#### Exiting
Listing owners can yank their listings from the market. This
removes the listing from the `Market` and will burn any minted listing reward
tokens.

```
function exit(bytes32 listingHash) external
```

#### Reserve
The `Market` holds with it an associated "reserve." Think of the
reserve as holding earnings from the data in the `Market` that belong
to all the `MarketToken` holders associated with the market. These
earnings can come from either query payments or from investor
purchases of `MarketToken`.  Investor class `MarketToken` holders are
allowed to withdraw earnings from the reserve by burning their
`MarketToken` holdings.

At present, the reserve is denominated in [`NetworkToken`](#network-token).


#### Investor and Owner Class
The `Market` will have two classes of `MarketToken` holders, investors and
data owners. Data owners can own particular listings in the `Market`.
However, they are not allowed to purchase new `MarketTokens` by calling
`Market.invest()` and they are not allowed to withdraw tokens from the
reserve by calling `Market.divest()`. Oppositely, an investor class
`MarketToken` holder is not allowed to own any listings in the market.

If a data owner wishes, they may convert to investor class by calling
`Market.convert_to_investor()`. This will surrender ownership of all
owned listings to the `Market`, and will convert the data owner to an
investor.  The transformation is not reversible at present; investors
cannot become data owners. Note that enforcement of this separation is
currently only performed at the level of Ethereum accounts; an
investor can always create a new Ethereum account and use that account
to become a data owner.


On the implementation end, an internal data structure will track
the class of each token holder in the `Market`. In addition, new token
holders will have to be entered into this internal data structure. Relevant methods:

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



Note that the linear form of the price curve above is not necessarily
set in
stone. It's likely that future iterations will allow users to choose alternate
forms of the price curve.

#### Paying for Computation 
Users may wish to run queries against the data in the `Market` or may
wish to construct machine learning models on this data. In order for
them to be authorized for such computation, they must first make a
payment via the `Market` contract.

The `Market` controls the payment layer for computation. Users who
wish to query the data listed in a data market must first make a
payment to `Market`. Any `Backend` associated with `Market` will check
that payments have gone through before allowing for queries.

Listing owners set an access cost for their listing (denominated in
`NetworkToken` wei). For listings which are owned by the market
itself,  the listing default price is set in the `Parameterizer`.

```
function set_access_cost(bytes32 listingHash, uint cost) external
```
Callable only by the listing owner. Sets the price (in `NetworkToken` wei) to access this listing

```
function get_access_costs(bytes32 listingHash) returns (uint)
```
Returns the access cost for a listing.

```
function get_backend_cost(string backend) public view returns (uint)
```
Returns the standard `Backend` cost for compute. In this version, there is only a set fee. A more refined pricing structure is still being actively researched.

```
function pay_for_compute() external
```
Users call this function to pay for one computational workload to be run on a `Backend`. Additional workloads will require additional calls to this function.
  

#### Datatrusts 
Each data market will maintain a list of authorized `Backend` systems.
A full vote of the council (#28) will be needed to add, remove, or
authorize `Backend` systems.

```
function get_backend_system() public view returns ([string])
```
Returns list of authorized backend systems for the market

```
function propose_backend_addition(string backend, address backend_address) external
```
Proposes the addition of a new authorized `Backend`. This addition
must be authorized by a vote of the council. The `string backend`
field is an external URL for the `Backend`. The `address
backend_address` is an Ethereum address owned by the `Backend`
operator.

```
function propose_backend_removal(string backend, address backend_address) external
```
Proposes that the specified `Backend` have its authorization revoked.
This removal must be authorized by a vote of the council.

#### Market Parameters
The `Market` is governed by a set of parameters controlled by the `Parameterizer`.

```
uint challengeStake
```
The stake (in `MarketToken`) needed to issue a challenge to a listing.

```
uint voteBy
```
The time (in seconds) that a poll should remain open. This controls the length
of the voting window in which council members can vote upon an `Market`
listing, challenge, or reparameterization.

```
uint quorum
```
The percent (whole number between 0 and 100) of the council which must vote in
favor of a `Market` modification for it to succeed.

```
uint dispensation
```

A percentage (whole number between 0 and 100) that is the fraction of `challengeStake` that the winner of a challenge receives.

```
uint conversionRate
```

The constant in the algorithmic price curve

```
uint conversionSlope
```
The slope in the algorithmic price curve.

```
uint listReward
```
The number of new `MarketToken` wei that are minted when a listing is listed.

#### Reparameterization

All market parameters can be changed with a council vote. The process of changing `Market` parameters is referred to as reparameterization.


### Epsilon Privacy Curve

![alt text][epsilon_price_curve]

[epsilon_price_curve]: epsilon_privacy_curve.png "Epsilon Price Curve"

The Epsilon price-curve is the tool used to price for
the privacy lost in a given query. Here, epsilon is a
technical parameter, adapted from the differential
privacy literature, which is a measure of the
information loss tied to a particular query. Each query
has an associated epsilon. Here are some possible APIs
for this feature.

- `Market.get_current_privacy_price(user)` returns the current price for purchasing additional privacy budget from the epsilon price curve. This depends on the current privacy epsilon used by the provided user.
- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

### Differential Privacy on Queries


![alt text][query_flow]

[query_flow]: QueryFlow.png "Query Flow"

![alt text][multi_market_join]

[multi_market_join]: Multi_Market_Join.png "Multi Market Join"


### Censorship Resistant Data Markets

It is possible to build data markets that are resistant to censorship efforts.

![alt text][censorship_resistant_data_markets]

[censorship_resistant_data_markets]: Censorship_Resistant_Data_Market.png "Censorship Resistant Data Markets"

