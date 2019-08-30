# CAPI: The Computable API

We've covered the core Computable contracts at some
depth in the last few chapters. However, we haven't
said too much detail about the actual datatrust
softare. We'll remedy that in this chapter.

Let's start from first principles. A Datatrust is a
system that is responsible for storing data off-chain.
As we dicussed in the [listings
chapter](../listings/index.html), a listing is uniquely
identified by its `listingHash`. It makes sense that
the datatrust should key its listing data by
`listingHash`. This suggests that the core of a
Datatrust is an off-chain key-value store that maps the
`listingHash` for a listing to the actual listing data.
Different implementations of a Datatrust can choose to
use different software backends to implement this
key-value store functionality.

We've mentioned briefly that the datatrust is a
key-value store that holds data off-chain. What else is
a datatrust responsible for doing? We saw a few of
these responsibilities in the [datatrust contract
chapter](../datatrust/index.html), but what else does a
datatrust have to do? Or put more generally, when is
some piece of software a datatrust? In particular what
are the external signals and messages that a datatrust
must respond to?

The brief answer to these questions is that a datatrust
is any piece of software which implements
[CAPI](https://github.com/computablelabs/capi) (the
Computable API). Briefly, CAPI is a set of endpoints
with associated functionality that must be implemented
by a datatrust. In the remainder of this chapter, we'll discuss the endpoints that a datatrust must implement to be compliant with CAPI

## Listings Endpoint

```
/listings
```

## Candidates Endpoint

```
/candidates
```

TODO: Add more detail on CAPI as it's fleshed out.

## Last Thoughts

In this chapter, we covered CAPI, the protocol which
governs the off-chain datatrust software. In the next
chapter, you'll learn more about the API libraries that
permit developers to talk to the on-chain smart
contracts. In particular, we'll discuss `computable.py`
and `computable.js`, the developer libraries we've
built to facilitate interactions with on-chain smart
contracts.

<div style="text-align: right"> <a href="../../docs/libraries">Next Chapter</a> </div>
