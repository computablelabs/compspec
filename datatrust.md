# The Datatrust

Alright, we've now talked through a whole set of the
smart contracts in the system. You've learned about
listings, the reserve, voting, and the tokens. Now onto
the datatrust.  There's something tricky we're going to
have to wrestle with. Fundamentally, a datatrust is a
piece of software that has to run off-chain (in a
[future chapter](../capi/index.html) you'll learn about
the specification for this off-chain software) since it
needs to store and serve potentially very large pieces
of data. How can we possibly bound the behavior of this software from the on-chain world?

In general, there's a gulf between the on-chain and
off-chain worlds in smart contract systems. It's hard
to reach between the two realms. In particular, it's
extraordinarily hard to prove facts about the off-chain
world from on-chain. This means that for now, the
on-chain `Datatrust` contract can only loosely control
the behavior of the off-chain datatrust software. We're
actively
[researching](https://forum.computable.io/t/towards-atomic-delivery-of-data/62)
cryptographic techniques to remedy this situation, but
for now this means that the datatrust is a semi-trusted
part of the system. This means that each data market
has to trust that the operator of its datatrust will
behave correctly. This isn't necessarily unreasonable
though. But it basically means the creator of a data
market should do some legwork to make sure the
datatrust is run securely, perhaps by running the
datatrust software themselves.

Let's return to the subject though. In the next few sections, we'll talk through some of on-chain guardrails we want to design around the datatrust.

## Voting to Add or Replace a Datatrust 

Each data market has only one datatrust. It makes sense that voting in this datatrust would require a stakeholder vote. If we can vote in a datatrust, we can also replace it, which serves as a primitive guard on datatrust behavior (behave badly enough and you'll be voted out). 

## Listing handling 
The Datatrust is responsible for storing the off-chain
data associated with listings. Listing candidates must
convey their off-chain candidate data to the Datatrust
node in order to be listed. The Datatrust acknowledges
receipt of this listing data by setting the `data_hash`
for a given listing on-chain. If this acknowledgement
is not registered on-chain, the listing candidate
cannot become a listing. This acknowledgement is done with the `Datatrust.setDataHash()` function:

```
@public
def setDataHash(listing: bytes32, data: bytes32):
  """
  @notice Allow a registered backend to set the data_hash for a given listing
  @param listing The hashed identifier of a current market listing
  @param data The hashed data held by the backend for said listing
  """
  assert msg.sender == self.backend_address
  self.data_hashes[listing] = data
```

This function simply cheks that the authorized
datatrust (which is at `self.backend_address`) called
it, then sets a hash for the listing. This is used by
`Listing.resovleApplication()` to check that the
datatrust has affirmed that the data for a listing
candidate was received before allowing it to be listed.

## Listing Utilization Records
The datatrust is responsible for maintaining a record of
which listings have been accessed by buyers. Remember
from the [listings chapter](../listings/index.html)
that a buyer of data only purchases a set number of
bytes. The datatrust is responsible for providing the
purchaser access to listings they desire, while
checking that the buyer has enough budget left. The
total number of bytes accessed from a given listing is
tracked on-chain in the `Datatrust.bytes_accessed`
field:

```
bytes_accessed: map(bytes32, uint256) # maps a listing to its currently unclaimed bytes accessed
```

There's another mapping `Datatrust.bytes_purchased` which tracks the total number of bytes purchased. This is updated by `Datatrust.requestDelivery` as you saw in the [listings chapter](../listings/index.html):

```
bytes_purchased: map(address, uint256) # maps user-address to total amount of bytes purchased
```


After a listing has been accessed by a buyer, the
datatrust has to report this on-chain by calling
`Datatrust.listingAccessed`

```
@public
def listingAccessed(listing: bytes32, delivery: bytes32, amount: uint256):
  """
  @notice Allow a backend to record the access of a listing, thus fulfulling part of a delivery.
  @dev Only a registered backend may call. Enforce that the claimed listing exists.
  @param listing The listing that was accessed
  @param delivery Which delivery object this access was for
  @param amount How many bytes were accessed
  """
  assert msg.sender == self.backend_address
  assert self.data_hashes[listing] != EMPTY_BYTES32
  # this can be claimed later by the listing owner, and are subtractive to bytes_purchased
  self.bytes_accessed[listing] += amount
  self.bytes_purchased[self.deliveries[delivery].owner] -= amount
  # bytes_delivered must eq (or exceed) bytes_requested in order for a datatrust to claim delivery
  self.deliveries[delivery].bytes_delivered += amount
```

This function call increments `self.bytes_accessed` for
this listing while decrementing `self.bytes_purchased`
by the same amount. The listing owner now has some
rewards they can claim. To do this claiming, they need
to call `Datatrust.bytesAccessedClaimed`:

```
@public
def bytesAccessedClaimed(hash: bytes32, fee: wei_value):
  """
  @notice Called by the Listing contract when a maker claims their listing access rewards
  @param hash The listing identifier
  @param fee Amount of ether token to transfer to the reserve
  """
  assert msg.sender == self.listing_address
  clear(self.bytes_accessed[hash]) # clear before paying
  self.ether_token.transfer(self.reserve_address, fee)
  # NOTE bytes accessed claimed event published by Listing contract
```

This function simply clears the backlog of
`bytes_accessed` for this listing owner and unlocks
their fair share of the purchase fee.

You might reasonably ask why a datatrust operator is
motivated to do all this work reporting which listings
were accessed. Well, the only way the datatrust gets
its fair payment is after it's finished reporting all
the listings that were accessed. To claim its payment,
the datatrust operator has to call
`Datatrust.delivered`:

```
@public
def delivered(delivery: bytes32, url: bytes32):
  """
  @notice Allow a backend to collect its payment.
  @dev We check that a backend has delivered, at least, the amount of bytes requested.
  NOTE: bytes_requested is the multiplier for backend payment.
  @param delivery Identifier of the delivery in question
  @param url A hash of the URL that the backend delivered to
  """
  assert msg.sender == self.backend_address
  owner: address = self.deliveries[delivery].owner
  requested: uint256 = self.deliveries[delivery].bytes_requested
  assert self.deliveries[delivery].bytes_delivered >= requested
  # clear the delivery record first
  clear(self.deliveries[delivery])
  # now pay the datatrust from the banked delivery request
  back_fee: wei_value = (self.parameterizer.getCostPerByte() * requested * self.parameterizer.getBackendPayment()) / 100
  self.ether_token.transfer(self.backend_address, back_fee)
  log.Delivered(delivery, owner, url)
```

This function checks that the full delivery has been
made, then pays out the datatrust operator. This
provides an on-chain check on the datatrust operator's
behavior and forces it to do some work before it gets
paid.

## Proposing a new datatrust.

An interested party can propose itself as a new
datatrust candidate by calling `Datatrust.register()`

```
@public
def register(url: string[128]):
  """
  @notice Allow a backend to register as a candidate
  @param url The location of this backend
  """
```
(TODO: There's currently a bug in register uncovered in audit. Add full code once bugfix is in place)

Calling this method triggers a vote. If the vote
passes, then this this party becomes the new datatrust
for the system. This means that the old datatrust (if
there was one), is essentially kicked out. 

## Last Thoughts

At this point, we've discussed all the major smart
contract systems in Computable, with the exception of
the `Parameterizer` which sets market parameters. We'll
hit this system in the next chapter.

<div style="text-align: right"> <a href="../../docs/parameters">Next Chapter</a> </div>
