# Listings
A data market holds a set of "listings". Each listing
corresponds to a chunk of data contributed by a single maker and managed within the data market. The listing is physicaally stored  
off-chain in the datatrust for this market. 

We reproduce the fields of the on-chain listing
structure (in `Listing.vy`) below.

```
struct Listing:
  owner: address
  supply: wei_value
```

Let's take a minute to walk through the fields of this
struct to explain how the `Listing` works. The
`Listing` is an on-chain record of a chunk of off-chain
data. Note that a listing has a very simple on-chain
footprint. It simply records the owner of the listing,
keyed by their Ethereum address, along with the
`supply` tied to the listing. The `supply` is a amount
of funds tied to this particular listing (TODO: clarify
this relationship).

You might wonder how two listings are differentiated
from each other if there's no unique identifier in the
struct. The answer to this is that the listings are
stored in a `map` that keys from the `listingHash` for
this listing to the actual struct.

```
listings: map(bytes32, Listing)
```

There is a last piece of data tied to each listing, the
`dataHash`. This is stored in the `Datatrust.vy`
contract.

```
data_hashes: map(bytes32, bytes32) # listing_hash -> data_hash
```

Once the datatrust has received the actual off-chain
listing data from the maker, it stores the `data_hash`
(conceptually the hash of the raw data for the listing,
but could be something else) within this map.

			- [What is a Datapoint?](#what-is-a-datapoint): Each listing corresponds to an off-chain "datapoint." This section defines precisely what a "datapoint" is. Briefly, a "datapoint" is just an arbitrary bytestring.
			- [Applying](#applying): Applying to add a listing to a data market
			- [Challenging](#challenging): Challenging an existing listing within a data market
			- [Exiting](#exiting): Yanking a listing from a data market. 


You might wonder how we can use these fields to tell whether a listing has been formally accepted (and is no longer a candidate). For this, we can use the convenient fact that Vyper defaults fields to 0 if not set:

```
@public
@constant
def isListed(hash: bytes32) -> bool:
  """
  @notice Return a boolean representing whether a Listing has been listed
  """
  return self.listings[hash].owner != ZERO_ADDRESS
```

Reading through this, if the listing's owner is not
`ZERO_ADDRESS`, then the listing has been accepted.

#### What is the off-chain part of a listing?
We haven't clearly specified what a listing is
precisely in the preceding material.  Part of the
challenge is that a "listing" will mean different
things for different markets. A record in an off-chain
SQL database is very different from an image file for a
deep learning application. For this reason, we say that
the off-chain part of a a listing is simply an
arbitrary bytestring.  This bytestring may correspond
to multiple "datapoints".  For example, the bytestring
may correspond to 10 SQL rows or to 50 images. This
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

