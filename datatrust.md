---
title: The Datatrust 
type: docs
---

#### Datatrusts 
Each data market will maintain a list of authorized
`Backend` systems.  A full vote of the council (#28)
will be needed to add, remove, or authorize `Backend`
systems.


```
function propose_backend_addition(string backend, address backend_address) external
```
Proposes the addition of a new authorized `Backend`.
This addition must be authorized by a vote of the
council. The `string backend` field is an external URL
for the `Backend`. The `address backend_address` is an
Ethereum address owned by the `Backend` operator.

```
function propose_backend_removal(string backend, address backend_address) external
```
Proposes that the specified `Backend` have its authorization revoked.
This removal must be authorized by a vote of the council.


## The Datatrust

The off-chain portion of the Computable protocol, the
Datatrust, is responsible for storing, querying and
delivering upon data. 

		- [Buying Data](#paying-for-computation): Each `Market` supports running computational workloads against the data in this market. Workloads are run on a `Datatrust` tied to the market and may include SQL queries and standard programs capable of executing in a standard Linux environment. Users have to pay for workloads before they may execute them on a `Datatrust`.
			- [Data utilization](#data-utilization): The market maintains track of how many times each listing has been requested by different queries.
		- [Datatrusts](#authorized-backends): The data listed in the data market is held off-chain in a `Datatrust`. A council vote is used to set authorized backend systems for this market.

### Datatrust Specification
A Datatrust is a system that is responsible for storing
data off-chain. As a first approximation, think of a
Datatrust as storing an off-chain key-value store that
maps `listingHash` to the actual listing data.

At present, a data market has only one datatrust. The
datatrust is currently a trusted entity in the
protocol. However, active research is under way to
reduce the trust requirements upon the datatrust. 

#### Storage
The Datatrust is responsible for storing the off-chain
data associated with listings. Listing candidates must
convey their off-chain candidate data to the Datatrust 
node in order to be listed. 

#### Computational Workloads 
The Computable protocol allows users to run workloads
on off-chain data. Payments for these workloads must be
made on-chain before workloads will be allowed to run.
In the current version of the protocol, security and
privacy guarantees are not enforced upon workloads, but
it is expected that future protocol iterations will
enforce such guarantees.

Users can expect that their workloads will run within a
fairly standard computing environment (some sandboxed
Linux environment likely) and that the raw data will be
exposed to the sandbox. Users will be able to run
standard SQL queries, and will also be able to use
standard Python machine learning tools to train models
and implement ETL pipelines. (Note that since the
sandbox will be some standard Linux, users are free to
use alternative pipelines).

The specification does not constrain `Backend`
implementers on their design choices, but to first
approximation, this feature should likely be
implemented by having new sandboxed compute nodes being
spun up dynamically to handle inbound queries. This
might be implemented for example by running the
computation within a docker container on a new EC2
node.

The user sends a string to the `Backend` holding the
script to be executed via the REST API. This will
usually be a bash script of some form coordinating the
execution of a program in the `Backend` Linux
environment. For ease, SQL might be handled in a
special case so the user doesn't need to set up a
suitable SQL environment via script each time.

As a note about implementation, a first design for the
workload support would be to have the `Backend` spin up
a preconfigured docker instance for each new user
script. The script would execute within this container.
Since container filesystems are ephemeral, scripts
wouldn't be capable of altering stored data within the
`Backend`.  Optionally, the container's network access
could be turned off so the workload can't directly
download `Backend` data.

Note furthermore that compute workloads may draw upon
data from multiple data markets. (The limitation of
course is that the `Backend` that the compute is
running on must be authorized for both data markets).

#### Coarse Data Utilization
The market is responsible for maintaining a record of
which listings have been accessed by which queries.
Doing this robustly is still an open [research
problem](#fine-grained-data-utilization). However, the
current `Backend` spec supports a coarse-grained record
by simply tracking the number of computations which
have been run on this data market. It's assumed that
each computation touched all listings. (This is
obviously wrong, but is useful for initial
implementation efforts).

```
function update_listings_accessed(uint num_workloads) external
```
This function can only be called by an authorized `Backend`. At present, only reports the number of queries run on this `Backend`.


#### REST API

The `Backend` is responsible for serving a number of endpoints. These endpoints are specified below. The syntax `Backend::ENDPOINT(input)` is used to specify that the `Backend` supports an `ENDPOINt` which expects `input`.

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

#### GET_SCHEMA

`Backend::GET_SCHEMA(market)` returns the schema associated with a
particular data market. Note that the schema is associated closely
with an individual market. In a future iteration of the design, the
schema may be an on-chain entity.

Note that the results of `Backend::GET_SCHEMA(market)` should specify
the bytestring format that is accepted by `Backend::ADD_DATAPOINT`.
The `Backend` is responsible for rejecting malformed bytestrings that
don't adhere to the schema.


#### GET_DATAPOINT
The call `Backend::GET_DATAPOINT(auth_token, market,
dataHash)` fetches the data point uniquely associated
with `dataHash` from the backend. It's worth pausing a
bit here to unpack this statement.  Recall that a
datapoint is a bytestring of arbitrary length. Why is
`dataHash` guaranteed to be uniquely identified with
this given bytestring? Recall that the hash function we
use is a cryptographic hash function. This means that
it's really hard to find collisions in cryptographic
hash functions (effectively impossible for a good
choice of cryptographic hash). So if the API call
returns a particular bytestring, the caller can
recompute the cryptographic hash locally and verify
that the hash of the returned value equals `dataHash`.

Let's take a brief digression and chat about
implementations. Although `dataHash` uniquely specifies
a datapoint, the `Backend` implementation must have an
implemented method to retrieve a given datapoint given
`dataHash`. The simplest possible implementation would
be for the `Backend` to store a flat file on disk of
form
```
[bytestring_1,...,bytestring_n]
```
Now let's suppose `H` is our cryptographic hash
function. Given `dataHash`, the backend could retrieve
the associated bytestring with asimple for-loop
```
for bytestring in [bytestring_1,...,bytestring_n]:
  if H(bytestring) == dataHash:
    return bytestring
```
This is very inefficient since it requires a full
`O(n)` traversal for every retrieval. Alternatively, we
could create a SQL table that stores this mapping for
us
```
hash, data
H(bytestring_1), bytestring_1
.
.
.
H(bytestring_n), bytestring_n
```
Then if we denote `hash` as the primary key for this
table, we can use a standard SQL lookup command to
retrieve the correct bytestring associated with
`dataHash`.


## Forwarding Looking Research

Features in this section are being actively researched
by the Computable team, but at present are not on the
roadmap for a particular Computable release. This is
expected to change as development matures, and it is
expected that all work in this section will eventually
migrate up into the concrete specifications part of
this document.

### Fine Grained Data Utilization

Tracking fine-grained data usage is necessary for fair
distribution of user rewards. It's expected that some
listings in a `Market` will be significantly more
valuable than others. These listings should receive
greater rewards than less valuable listings. To first
approximation, we can track data usage by keeping track
of which listings a particular computation touches on
the `Backend` side. The `Backend` could then report
these records to the on-chain contracts, which would
then update the usage records.

This gets tricky for more complex workloads for
example. A deep learning model will train on all data,
but some listings will contribute more to the value of
the model than others. Robustly attributing importance
to particular data points over others is still an open
question in machine learning with only a few research
prototypes out there.

