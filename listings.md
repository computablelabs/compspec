# Listings

This section introduces the core concept of a
"listing," which is central to the protocol.

		- [Listings](#listings): The basic elements of a data market.
			- [What is a Datapoint?](#what-is-a-datapoint): Each listing corresponds to an off-chain "datapoint." This section defines precisely what a "datapoint" is. Briefly, a "datapoint" is just an arbitrary bytestring.
			- [Applying](#applying): Applying to add a listing to a data market
			- [Challenging](#challenging): Challenging an existing listing within a data market
			- [Exiting](#exiting): Yanking a listing from a data market. 

A market holds a set of `Listings`. Each listing
corresponds to an element of the `Market` which is held
off-chain in some (possibly multiple) `Backend`
systems.  Newcomers to the market can call
`Market.apply()` to apply to have their listing added
to the market. A listing consists of an off-chain
datapoint (or datapoints) and an on-chain listing
structure. (We haven't defined "datapoint" here yet.)
We reproduce the fields of the on-chain listing
structure below.

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

Let's take a minute to walk through the fields of this
struct to explain how the `Listing` works. The
`Listing` is an on-chain record of a chunk of off-chain
data. The `dataHash` is the hash of the set of
off-chain data that this listing corresponds to. For
our purposes, this off-chain data is simply an
arbitrary blob (a bytestring of arbitrary length) that
is hashed down to a single `bytes32` value.

The `listed` boolean field specifies whether this
listing is officially listed or not in this given
market. The `owner` field is the market participant who
owns this listing. If this owner has converted to
investor class, ownership of the listing will be
transferred to the market itself and the `address` in
this field will be the market address.

The `supply` field is the number of `MarketToken` that
the listing proposer is willing to stake to see this
listing listed in the `Market`. This must exceed the
`minDeposit` that is demanded by the `Parameterizer`
tied to this market.  The purpose of this stake is to
reward challengers who remove useless listings from a
given market.

The `challenge` field tracks if there's an active
challenge to this `Listing` at present. `rewards`
tracks how many new `MarketToken` have been minted for
this `Listing`. Note that this field is only nonzero
for `Listings` which have successfully been listed.

 
#### What is a datapoint?
We haven't clearly specified what a "datapoint" is in
the preceding material.  Part of the challenge is that
a "datapoint" will mean different things for different
markets. A record in an off-chain SQL database is very
different from an image file for a deep learning
`Backend`. For this reason, we say that the "datapoint"
tied to a listing is simply an arbitrary bytestring.
This bytestring may correspond to multiple "logical
datapoints".  For example, the bytestring may
correspond to 10 SQL rows or to 50 images. This
batching might be crucial for efficiency, since the
transaction rate of Ethereum is not yet sufficient to
do bulk uploads of datasets otherwise.

#### Applying
Applying is the process by which a new listing is added to a data
market. To apply, a market participant computes the hash of their
off-chain data and proposes the addition of their data to the market
by invoking `Market.apply()`:

```
function apply(bytes32 listingHash, uint amount, string data) external
```

All applications trigger a vote on the new listing by
appropriate market stakeholders (either all
token-holders or the market council).  If a listing
vote is cleared, it is said to be listed. Note that
application is a *minting* event whereby new
`MarketTokens` are created. More detail on this can be
found in the section on minting.

#### Challenging
Challenging is the process by which a listing in a data
market can be challenged and potentially removed. A
challenge triggers a vote. If the challenge succeeds,
the challenged listing is de-listed from the data
market.  If the challenge fails, the challenging party
is penalized with a loss of stake (note that posting a
challenge requires placing `MarketToken` at stake).

Note that unlike a token curated registry, the council
receives no reward for voting upon a challenge. Only
the victor of the challenge receives a financial reward
which comes directly from the loser of of the
challenge.

#### Paying for Access
Users may wish to run queries against the data in the
`Market` or may wish to construct machine learning
models on this data. In order for them to be authorized
for such computation, they must first make a payment
via the `Market` contract.

Listing owners set an access cost for their listing
(denominated in `NetworkToken` wei). For listings which
are owned by the market itself,  the listing default
price is set in the `Parameterizer`.

```
function set_access_cost(bytes32 listingHash, uint cost) external
```
Callable only by the listing owner. Sets the price (in `NetworkToken` wei) to access this listing

```
function get_access_costs(bytes32 listingHash) returns (uint)
```
Returns the access cost for a listing.

