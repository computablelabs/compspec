---
title: The Datatrust 
type: docs
---

## The Datatrust

The off-chain portions of the Computable protocol are responsible for
storing, querying and computing upon data. All off-chain components
are called `Backend` systems, but there are multiple types of
backends. A trusted `Backend` is operated by an entity that is trusted
by the creator of a particular data market. The operator of this
trusted `Backend` is authorized by the data market's council with this
authority. By contrast, an untrusted `Backend` is not trusted by the
data market council and is *not* authorized to view unencrypted data
on the network. The untrusted `Backend`operator must use more advanced
techniques such as cryptography or trusted hardware enclaves to
perform computation since they may never see unencrypted data. At
present, the design of untrusted `Backends` is still a research
problem, and the remainder of this section will focus exclusively on
trusted `Backends`.

Note that the implementation of a `Backend` is not specified by this
document.  A `Backend` is any system that responds to the API
endpoints defined in this section. Notably, this means a `Backend` may
be closed source or proprietary.  In addition, a `Backend` may have
arbitrary hardware backing it. It could run on a single laptop or
could be a cluster of servers on the cloud. It could even use novel
proprietary hardware such as GPUs, TPUs or ASICs to power needed
workloads. These choices are left to the operator of the `Backend`.

### Datatrust Specification
A `Backend` is a system that is responsible for storing data
off-chain. Any `Market` contains a list of authorized `Backend`s which
hold the raw data associated with the `Market`. Currently, `Backends`
are assumed to be trustworthy. A trusted `Backend` is allowed to view
the unencrypted data that belongs to a `Market`.

`Backend` systems are responsible for user authentication, data
storage, encryption-at-rest, and computational workloads. In this
section, we start by describing these responsibilities qualitatively,
then walk through the precise REST API that the `Backend` must
support.

#### Authentication
The `Backend` should provide a mechanism for users to authenticate.
The first step of the authentication process will require the
`Backend` to consult with the on-chain `Market` and will likely
require the user to use a system like Metamask to prove their
identity. Once initial authentication is complete, users can be issued
an authentication token which they can use for a set time.

#### Storage
The `Backend` is responsible for storing the off-chain data associated
with listings. Listing candidates must convey their off-chain
candidate data to a `Backend` node in order to be listed. This is
enforced by the council vote; the council is responsible for querying
a `Backend` to verify that off-chain data for the candidate listing is
available before voting to list the candidate.

At the time of the main-net Computable release, the `Backend` should
be capable of supporting a TB scale datamarket. Note that a `Backend`
could support multiple data markets, so its total storage footprint
might be larger.

#### Encryption at Rest
The `Backend` is responsible for storing all off-chain data in a
fashion that's encrypted-at-rest. This means that plain text off-chain
data should never be stored on disk.

#### Computational Workloads 
The Computable protocol allows users to run workloads on off-chain
data. Payments for these workloads must be made on-chain before
workloads will be allowed to run. In the current version of the
protocol, security and privacy guarantees are not enforced upon
workloads, but it is expected that future protocol iterations will
enforce such guarantees.

Users can expect that their workloads will run within a fairly
standard computing environment (some sandboxed Linux environment
likely) and that the raw data will be exposed to the sandbox. Users
will be able to run standard SQL queries, and will also be able to use
standard Python machine learning tools to train models and implement
ETL pipelines. (Note that since the sandbox will be some standard
Linux, users are free to use alternative pipelines).

The specification does not constrain `Backend` implementers on their
design choices, but to first approximation, this feature should likely
be implemented by having new sandboxed compute nodes being spun up
dynamically to handle inbound queries. This might be implemented for
example by running the computation within a docker container on a new
EC2 node.

The user sends a string to the `Backend` holding the script to be
executed via the REST API. This will usually be a bash script of some
form coordinating the execution of a program in the `Backend` Linux
environment. For ease, SQL might be handled in a special case so the
user doesn't need to set up a suitable SQL environment via script each
time.

As a note about implementation, a first design for the workload
support would be to have the `Backend` spin up a preconfigured docker
instance for each new user script. The script would execute within
this container. Since container filesystems are ephemeral, scripts
wouldn't be capable of altering stored data within the `Backend`.
Optionally, the container's network access could be turned off so the
workload can't directly download `Backend` data.

Note furthermore that compute workloads may draw upon data from
multiple data markets. (The limitation of course is that the `Backend`
that the compute is running on must be authorized for both data
markets).

#### Coarse Data Utilization
The market is responsible for maintaining a record of which listings
have been accessed by which queries. Doing this robustly is still an
open [research problem](#fine-grained-data-utilization). However, the
current `Backend` spec supports a coarse-grained record by simply
tracking the number of computations which have been run on this data
market. It's assumed that each computation touched all listings. (This
is obviously wrong, but is useful for initial implementation efforts).

```
function update_listings_accessed(uint num_workloads) external
```
This function can only be called by an authorized `Backend`. At present, only reports the number of queries run on this `Backend`.


#### REST API

The `Backend` is responsible for serving a number of endpoints. These endpoints are specified below. The syntax `Backend::ENDPOINT(input)` is used to specify that the `Backend` supports an `ENDPOINt` which expects `input`.

- `Backend::AUTHENTICATE(user_credentials)`: Authenticates the given user. Returns an authorization token.
- `Backend::GET_SCHEMA(market)`: Returns the schema for the given market.
- `Backend::RUN_QUERY(auth_token, markets, query)`: Runs the given query against the given markets. The query is sent in a query file.
- `Backend::ADD_DATAPOINT(auth_token, market, data)`: Adds the given data point to the `Backend` along with necessary authorization.
- `Backend::REMOVE_DATAPOINT(auth_token, market, data)`: Removes the given data point from the `Backend`.
- `Backend::GET_DATAPOINT(auth_token, market, dataHash)`: Fetches the data point uniquely associated with `dataHash` from the `Backend`.
- `Backend::MARKETS_SUPPORTED()`: Returns the list of `Market`s which `Backend` supports.
- `Backend::GET_COMPUTE_COST(query)`: A call to the `Backend` via REST to get the cost for running specified query.
- `Backend::GET_EPSILON(query)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

We've represented these REST queries as methods, but remember that
these "methods" are actually REST API calls. The `auth_token` will be
contained in the request header and the other arguments will be passed
in the request body.

Let's dive into these methods in more detail.

#### AUTHENTICATE
Calling `Backend::AUTHENTICATE(user_credentials)` will authenticate
the given user with the particular Backend at hand. An authenticated
user will be able to submit queries directly to the Backend system
without needing to constantly communicate with the blockchain itself.
Note however, that the authentication process *will* check against the
blockchain to see if a particular user is authorized to access the
data on this Backend system. More specifically, 
`user_credentials` will be a nonce that is signed with the user's private key. 

Note in addition that the implementation of the specific
authentication system is not specified in this document. Different
backend creators may choose to create separate implementations. In
addition, note that each Backend system is responsible for
implementing its own authentication layer. However, since a given
Backend may serve many data markets, an authenticated user may be able
to easily query against multiple markets or run multi-market queries
on a given Backend system.


#### GET_SCHEMA

`Backend::GET_SCHEMA(market)` returns the schema associated with a
particular data market. Note that the schema is associated closely
with an individual market. In a future iteration of the design, the
schema may be an on-chain entity.

Note that the results of `Backend::GET_SCHEMA(market)` should specify
the bytestring format that is accepted by `Backend::ADD_DATAPOINT`.
The `Backend` is responsible for rejecting malformed bytestrings that
don't adhere to the schema.

#### RUN_QUERY

We don't specify in this document the structure of the query engine.
Conceivably, queries could include SQL, machine learning scripts, ETL
transformations and more. A future iteration of this document may specify the
precise forms of query strings accepted by this API.

#### ADD_DATAPOINT

The call `Backend::ADD_DATAPOINT(auth_token, market, data)` adds the
specified `data` to the specified `market`. Here `data` is an
arbitrary bytestring. The hash of this bytestring must be listed
within the on-chain data market contract as `listing.dataHash` for
some listing in order for `ADD_DATAPOINT` to succeed.

#### REMOVE_DATAPOINT
The call `Backend::REMOVE_DATAPOINT(auth_token, market, data)` removes
the specified `data` from the specified market. As before, `data` is
an arbitrary bytestring.  `data` must have previously been added in a
call to `ADD_DATAPOINT`. In addition, the hash of this bytestring must
appear as the `listing.dataHash` for a `listing` that has been removed
from the on-chain `Market` by its owner.  (Note that the on-chain
ledger is maintains the listings for removed data-points on-chain even
after they are no longer formally listed).


#### GET_DATAPOINT
The call `Backend::GET_DATAPOINT(auth_token, market, dataHash)`
fetches the data point uniquely associated with `dataHash` from the
backend. It's worth pausing a bit here to unpack this statement.
Recall that a datapoint is a bytestring of arbitrary length. Why is
`dataHash` guaranteed to be uniquely identified with this given
bytestring? Recall that the hash function we use is a cryptographic
hash function. This means that it's really hard to find collisions in
cryptographic hash functions (effectively impossible for a good choice
of cryptographic hash). So if the API call returns a particular
bytestring, the caller can recompute the cryptographic hash locally
and verify that the hash of the returned value equals `dataHash`.

Let's take a brief digression and chat about implementations. Although
`dataHash` uniquely specifies a datapoint, the `Backend`
implementation must have an implemented method to retrieve a given
datapoint given `dataHash`. The simplest possible implementation would
be for the `Backend` to store a flat file on disk of form
```
[bytestring_1,...,bytestring_n]
```
Now let's suppose `H` is our cryptographic hash function. Given
`dataHash`, the backend could retrieve the associated bytestring with
asimple for-loop
```
for bytestring in [bytestring_1,...,bytestring_n]:
  if H(bytestring) == dataHash:
    return bytestring
```
This is very inefficient since it requires a full `O(n)` traversal for
every retrieval. Alternatively, we could create a SQL table that
stores this mapping for us
```
hash, data
H(bytestring_1), bytestring_1
.
.
.
H(bytestring_n), bytestring_n
```
Then if we denote `hash` as the primary key for this table, we can use
a standard SQL lookup command to retrieve the correct bytestring
associated with `dataHash`.

#### MARKETS_SUPPORTED
The call to `Backend::MARKETS_SUPPORTED()` returns a list of the
markets which are supported on this Backend. Note that no
authentication is needed for this call.

#### GET_COMPUTE_COST

The `Backend::GET_COMPUTE_COST(script)` API call provides a Backend
system's estimated cost to run the script. Note that the method in
which a particular Backend system estimates these costs is not
specified in this document. Different Backend systems may have
different cost estimation methods.  For example, a Backend could use
current price estimates on cloud providers to set cost, or perhaps its
own internal statistics.

#### GET_EPSILON
The call to `Backend::GET_EPSILON(query)` computes the differential
privacy parameter epsilon that is associated with this given query.
Note that this call may sometimes fail, when it is not possible to
compute epsilon for the query at hand.


## Forwarding Looking Research

Features in this section are being actively researched by the
Computable team, but at present are not on the roadmap for a
particular Computable release. This is expected to change as
development matures, and it is expected that all work in this section
will eventually migrate up into the concrete specifications part of
this document.

### Fine Grained Data Utilization

Tracking fine-grained data usage is necessary for fair distribution of
user rewards. It's expected that some listings in a `Market` will be
significantly more valuable than others. These listings should receive
greater rewards than less valuable listings. To first approximation,
we can track data usage by keeping track of which listings a
particular computation touches on the `Backend` side. The `Backend`
could then report these records to the on-chain contracts, which would
then update the usage records.

This gets tricky for more complex workloads for example. A deep
learning model will train on all data, but some listings will
contribute more to the value of the model than others. Robustly
attributing importance to particular data points over others is still
an open question in machine learning with only a few research
prototypes out there.

### Pricing for Backend Payments

How should a `Backend` set its required payment for a computational workload?

### Query Rake 

For query payments that come in, a portion of the query payment (the
"rake") is paid into the reserve, a portion is paid to listing owners,
and a portion is paid to the `Backend`. It isn't yet clear how this
payment should be split up amongst these three participants. Economic
modeling will have to be done to understand the consequences of this
split.

Very simple (likely wrong) split:

- The owner of a listing is paid the access cost they set with `Market.set_access_cost(listing)`
- The `Backend` system is paid the compute cost they set with `Market.set_query_compute_cost(query)`
- The reserve is paid the privacy cost `Market.get_privacy_cost(query)`


