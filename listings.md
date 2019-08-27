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
Applying is the process by which a new listing is added
to a data market. To apply, a maker 
calls the following method on-chain:

``
@public
def list(hash: bytes32):
  """
  @notice Allows a maker to propose a new listing to a Market, creating a candidate for voting
  @dev Listing cannot already exist, in any active form. Owner not set here as it's an application
  @param An Ethereum hash (keccack256) serving as a unique identifier for this Listing when hashed
  """
  assert not self.voting.isCandidate(hash) # not an applicant
  assert self.listings[hash].owner == ZERO_ADDRESS # not already listed
  self.voting.addCandidate(hash, APPLICATION, msg.sender, self.parameterizer.getStake(), self.parameterizer.getVoteBy())
  log.Applied(hash, msg.sender)
```

This function does a few basic sanity checks, then puts
this candidate listing up for a vote. If the vote clears,s the candidacy is resolved and the candidate is listed and its `owner` field is set.

#### Challenging
Challenging is the process by which a listing in a data
market can be challenged and potentially removed. A
challenge triggers a vote. If the challenge succeeds,
the challenged listing is de-listed from the data
market.  If the challenge fails, the challenging party
is penalized with a loss of stake (note that posting a
challenge requires placing `MarketToken` at stake).

```
@public
def challenge(hash: bytes32):
  """
  @notice Challenge a current listing, creating a candidate for voting if it should remain
  @dev Must actually be listed, and not already challenged.
  @param hash The identifier for the listing being challenged
  """
  assert self.listings[hash].owner != ZERO_ADDRESS
  assert not self.voting.isCandidate(hash) # assure not already challenged
  self.voting.addCandidate(hash, CHALLENGE, msg.sender, self.parameterizer.getStake(), self.parameterizer.getVoteBy())
  log.Challenged(hash, msg.sender)
```

Note that unlike a token curated registry, the council
receives no reward for voting upon a challenge. Only
the victor of the challenge receives a financial reward
which comes directly from the loser of of the
challenge.

### Exiting
The owner of a listing can remove it from the market at any time by calling the following method:

```
@public
def exit(hash: bytes32):
  """
  @notice Allows the owner of a Listing to remove it from the market
  @dev Returns all Listing Supply to the owner. Burns all minted/rewarded funds associated with the Listing.
  Must actually be listed, and must not be involved in a Challenge
  @param hash Listing identifier
  """
  assert self.listings[hash].owner == msg.sender
  assert not self.voting.isCandidate(hash)
  self.removeListing(hash)
```

There are no controls or barriers on removing listings.
The listing owner maintains full control over their
listing in a data market.

### Purchasing access to a listing 
Users may wish to purchase access to a listing. They may do so by calling the `requestDelivery` method in `Datatrust.vy`:

```
@public
def requestDelivery(hash: bytes32, amount: uint256):
  """
  @notice Allow a user to purchase an amount of data to be delivered to them
  @param hash This is a keccack hash recieved by a client that uniquely identifies a request. NOTE care should be taken
  by the client to insure this is unique.
  @param hash A unique hash generated by the client used to identify the delivery
  @param amount The number of bytes the user is paying for.
  """
  assert self.deliveries[hash].owner == ZERO_ADDRESS # not already a request
  total: wei_value = self.parameterizer.getCostPerByte() * amount
  res_fee: wei_value = (total * self.parameterizer.getReservePayment()) / 100
  self.ether_token.transferFrom(msg.sender, self, total) # take the total payment
  self.ether_token.transfer(self.reserve_address, res_fee) # transfer res_pct to reserve
  self.bytes_purchased[msg.sender] += amount # all purchases by this user. deducted from via listing access
  self.deliveries[hash].owner = msg.sender
  self.deliveries[hash].bytes_requested = amount
```

There's a number of things going on here, but the basic
idea is that the purchaser has to pay up-front. (TODO:
Add more about the delivery flow) 
