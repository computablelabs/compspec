# Listings
A data market holds a set of "listings". Each listing
corresponds to a chunk of data contributed by a single
maker and managed within the data market. The listing is
physically stored  off-chain in the datatrust for this
market.

As we will see soon, listings are governed by the
on-chain voting system we discussed in the last
chapter. Adding a new listing, and challenging someone
else's listing to have it listing both require on-chain
votes. This permits for market stakeholders to actively
manage the set of available listings. It's worth noting
that listings require some off-chain governance as
well, since each listing is tied to an off-chain
(potentially large) chunk of data. For this reason, the
datatrust plays a role in verifying that makers have
actually submitted data for listing candidates before
they can be listed. Don't worry if you didn't digest
all that just yet. We'll say more in the rest of this
chapter.

## On-chain Listings

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
of funds tied to this particular listing. In
particular, if a listing candidate is approved and
listed in the market, the `listing_reward` of minted
`MarketToken` is transferred into `supply`.

Note here that both listing candidates and listings use
this same struct. A listing candidate simply has its
owner set to 0 to indicate null ownership. Once the
candidate is voted in and accepted, its owner is set,
indicating that it has been listed in the market. We
can see this in the `Listing.isListed` function:

```
@public
@constant
def isListed(hash: bytes32) -> bool:
  """
  @notice Return a boolean representing whether a Listing has been listed
  """
  return self.listings[hash].owner != ZERO_ADDRESS
```

If the listing's owner is not `ZERO_ADDRESS`, then the
struct corresponds to a listing candidate..

You might wonder how two listings are differentiated
from each other if there's no unique identifier in the
struct. The answer to this is that the listings are
stored in a `map` that keys from a unique `listingHash`
for this listing to the actual struct.

```
listings: map(bytes32, Listing)
```

This `listingHash` is used as the unique identifier for
the listing. It's how you can retrieve the listing from
a datatrust (if properly authorized; more on this
later).  There is a last piece of data tied to each
listing, the `dataHash`. This is stored in the
`Datatrust.data_hashes` field.

```
data_hashes: map(bytes32, bytes32) # listing_hash -> data_hash
```

Once the datatrust has received the actual off-chain
listing data from a maker, it stores the `data_hash`
(conceptually think of this as the hash of the raw data
for the listing, but could be something else) within
this map.

## What is the off-chain part of a listing?
So far, we've introduced the on-chain struct that
specifies listings and listing candidates. But where is
the actual data tied to the listing? Does it have any
specific format? Where does the data live?

Part of our challenge is that a "listing" will mean
different things for different markets. A biological
expexperiment outcome in a drug discovery data market
is very different from an image file for a vision data
market. For this reason, we say that the off-chain part
of a a listing is simply an arbitrary bytestring.  This
bytestring may correspond to multiple "datapoints" in
the usual sense.  For example, the bytestring may
correspond to 10 SQL rows or to 50 images. This
batching might be crucial for efficiency, since the
transaction rate of Ethereum is not yet sufficient to
do bulk uploads of datasets otherwise.

The listing lives in the datatrust associated to the
data market. As part of the process for getting a
listing candidate accepted, the maker is responsible
for sending the listing data to the datatrust for that
market. (You'll learn more in a [future
chapter](../capi/index.html) about the mechanics of
actually sending this data).

A buyer who purchases access to a listing similarly
needs to interact with a datatrust to download the
listing data.

## Applying
Applying is the process by which a new listing
candidate is proposed for addition to a data market. To
apply, a maker calls the following method on-chain:

```
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
this listing candidate up for a vote. If the vote
clears, the candidacy is resolved and the candidate is
listed when its `owner` field is set.

## Challenging
Challenging is the process by which a listed listing
can be challenged and potentially removed. A challenge
triggers a vote. If the challenge succeeds, the
challenged listing is de-listed from the data market
(its owner is set to 0).  If the challenge fails, the
challenging party is penalized with a loss of stake
(note that posting a challenge requires placing
`MarketToken` at stake).

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

Note that unlike a token curated registry, stakeholders 
receives no reward for voting upon a challenge. Only
the victor of the challenge receives a financial reward
which comes directly from the loser of of the
challenge.

## Exiting
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

You might wonder, couldn't this enable fraud? That is,
a malicious maker could repeated re-submit the same
listing, get it accepted, exit and restart. The
datatrust can prevent this attack to some degree
(perhaps by checking the hashes of newly submitted
listing candidates against previously submitted
listings), but wouldn't some on-chain defense mechanism
be better? The issue is that this would make it harder
for an honest maker to pull out data that they don't
want exposed. The protocol leans towards the rights of
makers in ambiguous situations, so we chose to not have
excessive friction for maker exits.

## Purchasing access to a listing 
Users may wish to purchase access to a listing. They
may do so by calling the `Datatrust.requestDelivery`:

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
idea is that the purchaser has to pay up-front for
access to a certain number of bytes. Once this payment
is made, the buyer can communicate with the datatrust
(details in a  [future chapter](../capi/index.html)) to
obtain access to some listings. The datatrust will keep
track of the on-chain state, and will not permit access
to more listings than the buyer has paid for.

There's a number of things in this code snippet we're
not yet explaining in detail. You might wonder what
this `self.reserve_address` is for example. You'll
learn in the [next chapter](../reserve/index.html).

## Last Thoughts

You might not have digested everything in this chapter.
There's a lot going on with listings since they're the
central entity of the data market. Keep reading and
refer back as you're confused. We'll have more examples
and discussion that will help clarify in the upcoming
chapters.

[Next Chapter](../reserve/index.html)
