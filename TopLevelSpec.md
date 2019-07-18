# The Computable Protocol

This document contains both concrete engineering specifications and
more forward looking research projections. To make the separation
clear, we've grouped the concrete engineering specifications in one
section and research projections in another. It's expected that the
research projections will migrate over time into the concrete engineer
section, but the precise timeline for this migration isn't yet known.
Subsections of the concrete portion of the spec are tagged with the
version(s) in which these features appear.

This document is a living, versioned specification. As understanding
of the core aspects of the Computable protocol grows, this document
will be updated accordingly.


## Top Level Specification

This section provides a high level roadmap of the full protocol with
links to more detailed specifications in subsequent sections. Various
sections are tagged with the Computable protocol version in which they
become available. Specifications for future versions are still in flux
and may change.

- [Concrete Engineering Specification](#concrete-engineering-specification): Features within this portion of the spec are tied to a specific version of the Computable protocol and are on the roadmap for the main network launch.
  - [On-chain smart contracts](#on-chain-components):
    - [`MarketToken`](#market-token): A mintable and burnable token. Each `Market` has its own `MarketToken`
        - [Minting and Burning Mechanics](#minting-and-burning-mechanics): Market tokens are minted when either new data is added, existing data is queried, or new investment is added to reserve. Market tokens are burned when data is removed or investment is withdrawn.
		- [Voting](#voting): Critical decisions within a market are performed by vote of the council (significant stake-holders). These include validation of new data, challenges to fraudulent data and changes to market structure. An ownership threshold `T_council` is imposed for franchise. The threshold will be set upon construction.
		- [Listings](#listings): The basic elements of a data market.
			- [What is a Datapoint?](#what-is-a-datapoint): Each listing corresponds to an off-chain "datapoint." This section defines precisely what a "datapoint" is. Briefly, a "datapoint" is just an arbitrary bytestring.
			- [Applying](#applying): Applying to add a listing to a data market
			- [Challenging](#challenging): Challenging an existing listing within a data market
			- [Exiting](#exiting): Yanking a listing from a data market. 
		- [Market Reserve](#market-reserve) [v0.2]: The reserve is the "bank account" associated with a given `Market`. 
		- [Algorithmic Price Curve](#algorithmic-price-curve) [v0.2]: Controls the price at which new investors may invest in market. Investor funds are deposited in reserve and new market token is minted accordingly.
		- [Buying Data](#paying-for-computation): Each `Market` supports running computational workloads against the data in this market. Workloads are run on a `Datatrust` tied to the market and may include SQL queries and standard programs capable of executing in a standard Linux environment. Users have to pay for workloads before they may execute them on a `Datatrust`.
			- [Data utilization](#data-utilization): The market maintains track of how many times each listing has been requested by different queries.
		- [Datatrusts](#authorized-backends): The data listed in the data market is held off-chain in a `Datatrust`. A council vote is used to set authorized backend systems for this market.
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


## Concrete Engineering Specification

This portion of the specification document deals with the part of the
protocol that is currently implemented. Specific subsections are
tagged 

### On Chain Components

The on-chain components of the protocol control economics and access
control.  If a user wants to gain access to a particular dataset (in a
particular data market), or if a user wants to invest in a particular
data market, they have to seek on-chain authorization. If a user wants
to pay for queries, this is also done off-chain. The advantage of this
structure is that payments and authorization can be handled securely
by secure on-chain contracts.

At present, on-chain contracts are implemented as Ethereum Solidity
contracts.  This does mean that the transaction/authorization speed is
limited by the current transaction speed on Ethereum.

#### Market 
The `Market` is the central contract that governs the behavior of a
data market. Here's a brief overview of the core functionality of the `Market`.

- The `Market` has an associated `MarketToken`. This `MarketToken` is created upon construction of the market. This token is minted and burned by various `Market` operations.The [`MarketToken`](#market-token) is itself a mintable and burnable ERC20 token.
- The `Market` holds a [reserve](#market-reserve) to the Market. This reserve holds `NetworkToken` that is paid in by investors who want to take positions in market and will pay out to people who want to exit market. Investors can purchase `MarketToken` by paying `NetworkToken` into the reserve. They can withdraw `NetworkToken` from the reserve by burning their `MarketToken` holdings.
- The `Market` has an "algorithmic price curve" that provides an automatic conversion rate from `NetworkToken` to `MarketToken`. The price curve is used by `Market.invest()` to determine the current conversion rate. The current conversion rate depends on the current size of the reserve.
- The `Market` supports a payment layer for computational workloads against the underlying data market. Payments must be made via `Market` before any `Backends` will accept queries.
- `MarketToken` holders in `Market` belong to one of two classes, data owner and investor. Only data owners can own listings in the market, and only investors have the right to withdraw from the reserve. A data owner can convert into investor class by giving up ownership of their listings. 

#### Market Token
`MarketToken` is a mintable and burnable ERC20 token. The
`MarketToken` is tied to a particular `Market` and is created when the
`Market` is created. Note the contrast with token curated registries,
which don't hold a mechanism for minting and burning their associated
token.

As with the `NetworkToken`, the `MarketToken` is denominated in
"market wei".  As with ETH, a "market wei" denominates 1/10^18 of a
`MarketToken`. Using wei units throughout prevents rounding error
propagation and keeps contracts simple.

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
Major decisions in the `Market` are made by token holder vote.
These decisions include which new listings should be added to the `Market`,
which challenged listings should be removed, and what changes should be made to
the `Market` parameters.

All votes are made by the "Council." The council is a subset of the
token-holders in the `Market` who hold a large fraction of the total number of
`MarketTokens`.  A threshold `T_council` will be imposed, and only
`MarketToken` holders who hold more than `T_council` units of `MarketToken`will
be allowed to vote.  Market participants who hold more than `T_council` units
of `MarketToken` are referred to as council members. Non-council members will
not be allowed to vote on market actions in this scheme. Note that since
`MarketToken` is burned and minted dynamically, the council can and will change
over time.

The votes here are *not* stake-weighted. All council members have precisely one
vote. So a council member with `5*T_council` and another council member
`1.1*T_council` `MarketTokens` have the same voting power. In addition, all
council votes at present are cast publicly with no lock-commit-reveal scheme.
This allows for the implementation of a simple voting mechanism with smaller
attack surface.


#### Listings 
**(versions 0.2,0.3):** A market holds a set of `Listings`. Each listing corresponds to an element of the
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

## Off Chain Systems [v0.2, v0.3]

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



### Epsilon Privacy Curve

![alt text][epsilon_price_curve]

[epsilon_price_curve]: epsilon_privacy_curve.png "Epsilon Price Curve"

The Epsilon price-curve is the tool used to price for the privacy lost
in a given query. Here, epsilon is a technical parameter, adapted from
the differential privacy literature, which is a measure of the
information loss tied to a particular query. Each query has an
associated epsilon. Here are some possible APIs for this feature.

- `Market.get_current_privacy_price(user)` returns the current price for purchasing additional privacy budget from the epsilon price curve. This depends on the current privacy epsilon used by the provided user.
- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

### Differential Privacy on Queries


![alt text][query_flow]

[query_flow]: QueryFlow.png "Query Flow"

### Untrusted Backend

An untrusted `Backend` is never allowed to view unencrypted data
belonging to the data market. This means that all computation
performed within an untrusted backend must either use cryptographic
techniques such as garbled circuits or homomorphic computation, or
must use trusted hardware enclaves like Intel SGX.  We are actively
researching the design of untrusted `Backend` systems, but currently
lack the clarity to place them on the engineering roadmap.

![alt text][multi_market_join]

[multi_market_join]: Multi_Market_Join.png "Multi Market Join"


## Case Studies

Thus far, we've discussed the Computable protocol in the abstract. In
this section, we walk through a few case studies of different types of
data markets that can be constructed using the Computable protocol.

### Censorship Resistant Data Markets

It is possible to build data markets that are resistant to censorship efforts.

![alt text][censorship_resistant_data_markets]

[censorship_resistant_data_markets]: Censorship_Resistant_Data_Market.png "Censorship Resistant Data Markets"

## Attacks

In this section, we list a number of known attacks on the protocol and talk through defenses against these attacks.

### Data Flood
In this attack, malicious actors attempt to flood a `Market` with low
quality listings. This attack is mitigated by the enforced council
vote needed for listings to be listed

### Council DDOS
Malicious attackers flood the council with a glut of low quality candidate listings.

TODO: Why are we safe against this attack?

### No Off Chain Data
The "No Off Chain Data" attack occurs when a candidate is listed on a
`Market` without its off-chain data being transmitted to a `Backend`.
This attack is defended against by the council vote. The council is
responsible for querying a `Backend` to verify that off-chain data has
been transmitted before voting to list the candidate
