# The Datatrusts
A Datatrust is a software system that is responsible
for storing data and coordinating with on-chain
permissions layers.  Put another way, the Datatrust is
the off-chain portion of the Computable protocol,
responsible for storing data, and delivering it upon
purchase. 

Each data market maintains an authorized Datatrust
on-chain.  A stakeholder vote is needed to authorize or
remove a datatrust. 

## Datatrust Implementation Specification
A Datatrust is a system that is responsible for storing
data off-chain. In particular, the core of a Datatrust
is an off-chain key-value store that maps the
`listingHash` for a listing to the actual listing data.
Different implementations of a Datatrust might choose
to use different software backends to implement this
key-value store functionality.

At present, each data market has only one datatrust.
The datatrust is currently a trusted entity in the
protocol. There does not exist a way to penalize a
Datatrust beyond removing its authorization by a
council vote. However, active research is under way to
reduce the trust requirements upon the datatrust. 

## Listing handling 
The Datatrust is responsible for storing the off-chain
data associated with listings. Listing candidates must
convey their off-chain candidate data to the Datatrust
node in order to be listed. The Datatrust acknowledges
receipt of this listing data by setting the `data_hash`
for a given listing on-chain. If this acknowledgement
is not registered on-chain, the listing candidate
cannot become a listing.

## Listing Utilization Records
The market is responsible for maintaining a record of
which listings have been purchased. 
```
@public
def listingAccessed(listing: bytes32, delivery: bytes32, amount: uint256):
```
This function can only be called by an authorized
Datatrust. These reported access numbers are used to
mint utilization rewards for makers.

## Proposing a new datatrust.

An interested party can propose itself as a new
datatrust by calling `Datatrust.register()`

```
@public
def register(url: string[128]):
  """
  @notice Allow a backend to register as a candidate
  @param url The location of this backend
  """
```

Calling this method triggers a vote. If the vote
passes, then this this party becomes the new datatrust
for the system.

[Next Chapter](../parameters/index.html)
