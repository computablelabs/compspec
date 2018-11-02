# The Computable Protocol

The Computable protocol creates a decentralized data market.  The global
network is made up of many individual `Market` instances. Each `Market`
conceptually holds a single dataset and is created and controlled by the owners
of the dataset. These owners could correspond to existing organizations, or
could be a decentralized set of interested parties. The coordination and access
control for these individual `Market` instances is coordinated by a set of
smart contracts. Each `Market` allows for a set of associated financial
operations. These operations allow interested parties to invest in a particular
`Market` or pay for the ability to query the data associated with that
`Market`. To facilitate these transactions, each `Market` has a unique
associated `MarketToken`. It is also possible for transactions to involve
multiple `Market` instances. For these purposes, the Computable network has a
global `NetworkToken` which mediates cross-`Market` interactions.

Everything described above is implemented in a set of smart contracts which
currently live on the Ethereum blockchain. However, it's important to note that
data itself can't live on smart contracts. For one, datasets can be very large
(gigabytes, terabytes, petabytes, exabytes or more). It would be infeasible to
store such large collections of data on existing smart contract systems. For
this reason, data lives "off-chain" in `Backend` systems. A `Backend` is
software system that is responsible for storing data and coordinating with
on-chain permissions layers. Note that many possible `Backend` implementations
are possible by different vendors or groups, so long as each implementation
responds to the API specified within this document. 

This document is a living, versioned specification. As understanding of the
core aspects of the Computable protocol grows, this document will be updated
accordingly.

![alt text][protocol_flowchart]

[protocol_flowchart]: Protocol_Flowchart.png "Protocol Flowchart"


## Top Level Specification

This section provides a high level roadmap of the full protocol with links to more detailed specifications in subsequent sections. Various sections are tagged with the Computable protocol version in which they become available. Specifications for future versions are still in flux and may change.

- [On-chain smart contracts](#on-chain-components):
  - [`MarketFactory`](#market-factory) [v0.3]: The top level entry point to create a new market and associated token.
  - [`NetworkToken`](#network-token) [v0.2]: The top level token for the entire network.
  - [`Market`](#market) [v0.2] The top level contract for a given data market.
    - [`MarketToken`](#market-token) [v0.2]: A mintable and burnable token. Each `Market` has its own `MarketToken`
      - Minting and Burning mechanics: Market tokens are minted when either new data is added, existing data is queried, or new investment is added to reserve. Market tokens are burned when data is removed or investment is withdrawn.
    - [Validation](#####)[v0.1]: How new listings are added to market.
      - [Voting](#voting) [v0.1, v0.3]: Critical decisions within a market are performed by vote of interested stake holders. These include validation of new data, challenges to fraudulent data and changes to market structure.
        - [All token holder vote](#all-token-holder) [v0.1]: At present all holders of `MarketToken` vote on decisions.
        - [Council member vote](#council-member-vote) [v0.3]: an ownership threshold `T_council` is imposed for franchise. The threshold will be set upon construction.
      - [Listings](#listings): The basic elements of a data market.
        - [Applying](#applying): Applying to add a listing to a data market
        - [Challenging](#challenging): Challenging an existing listing within a data market
        - [Editing](#######): Editing a listing within a data market.
    - [Market Reserve](#market-reserve) [v0.2]: The reserve is the "bank account" associated with a given `Market`. 
      - The [algorithmic price curve](#algorithmic-price-curve) [v0.2]: Controls the price at which new investors may invest in market. Investor funds are deposited in reserve and new market token is minted accordingly.
      - [Investor and data owner class tokens](#investor-and-owner-class) [v0.2]: Holders of market token are investor class or data owner class. Investor class tokens can't own any listings in the market, but have right to withdraw funds from reserve by burning their tokens. Data owner class tokens can own listings in market, but can't withdraw funds from reserve. 
    - [Queries](#queries) [v0.3]: Each `Market` supports queries against the data in this market. Queries are run on a `Backend` tied to the market and can be specified in a supported [query language](#query-language)
      - [Query Pricing](#query-pricing) [v0.3]: Users have to pay to run queries. This pricing structure has to reward the various stakeholders including listing owners (data), backend system owners (compute), and the market itself (investors)
      - [Query Rake](#query-rake) [v0.3]: What fraction of the payment goes to each stake holder?
      - [Data utilization](#data-utilization) [v0.3]: The market maintains track of how many times each listing has been requested by different queries.
    - [Authorized Backends](#authorized-backends) [v0.2]: The data listed in the data market is held off-chain in a `Backend`. A council vote is used to set authorized backend systems for this market.
- [Off-chain storage and compute systems](#off-chain-systems) [v0.2, v0.3]
  - [Backend Systems](#backend-specification) [v0.2, v0.3]: A `Backend` is responsible for securely storing data off-chain and allowing authorized users to query this data. Note that a `Backend` may serve multiple markets, and that a `Market` may have multiple backends. The `Backend` is an off-chain system that responds to the API specified in this document, and which understands how to interact with the on-chain Computable contracts.
    - [REST API](#rest-api) [v0.3]: The `Backend`must respond to a defined set of REST API commands to perform actions such as authentication, data addition and removal, and query handling 
    - [Query Language](#query-language) [v0.3]: Queries must be provided to `Backend` in query files that are written in a supported query language.
- [Case Studies](#case-studies) We consider a few case studies of interesting data markets that can be constructed with the Computable protocol in this section.
  - [Censorship Resistant Data Market](#censorship-resistant-data-markets) The Computable protocol allows for the construction of data markets that are resistant to censorship efforts.


## On Chain Components

The on-chain components of the protocol control economics and access control.
If a user wants to gain access to a particular dataset (in a particular data
market), or if a user wants to invest in a particular data market, they have to
seek on-chain authorization. If a user wants to pay for queries, this is also
done off-chain. The advantage of this structure is that payments and
authorization can be handled securely by secure on-chain contracts.

At present, on-chain contracts are implemented as Ethereum Solidity contracts.
This does mean that the transaction/authorization speed is limited by the
current transaction speed on Ethereum.

### Market Factory [v0.3]
The `MarketFactory` contract is responsible for creaking new data markets and will store a list of available data markets.

- `MarketFactory.create_data_market()`: Constructs and launches a new data market. This is the only public way to create a new data market. There are a number of arguments needed in this constructor.
Each data market has an associated token with it. `create_data_market()` should pass in necessary information to initialize this token. It might make sense to pass in a list of `[address_1: allocation_1,...,address_n:allocation_n]` of initial token allocations to `create_data_market()`. The Market token would be initialized with this initial spread of market token.
- `MarketFactory.T_council`: Council membership threshold fraction #28
- `MarketFactory.T_util`: Number of tokens minted when a listing is queried #31
- `MarketFactory.T_submit`: Number of tokens minted when a new listing is listed #31
- `MarketFactory.get_list_of_data_markets()`: Returns a list of available data markets.

### Network Token [v0.2]
The `NetworkToken` is the central token that powers the Computable network. It
is used by the [MarketFactory](MarketFactory.md) to perform operations and is
used to pay for queries executed by a `Backend`. The `NetworkToken` is implemented
by a `StandardToken` (ERC20) for now. 

### Market [v0.2]

The `Market` is the central contract that governs the behavior of a data
market. The current `Market` implementation has evolved from a `Registry`
implementation, but differs in a number of critical ways:

- The `Market` has an associated `MarketToken`. This `MarketToken` is created upon construction of the market. This token is minted and burned by various `Market` operations.
  - The [`MarketToken`](MarketToken.md) is a mintable and burnable ERC20 token.
- The `Market` holds a [reserve](#market-reserve) to the Market. This reserve holds `NetworkToken` that is paid in by investors who want to take positions in market and will pay out to people who want to exit market.
  - `Market.invest(amount)` is a new method that allows an investor to enter the market.
  - `Market.divest()` allows any token holder to exit the market
- The market holds an "algorithmic price curve" that provides an automatic conversion rate from `NetworkToken` to `MarketToken`. The price curve is used by `Market.invest()` to determine the current conversion rate.
  - `Market.get_current_investment_price()` returns the current NetworkToken/MarketToken exchange rate from the price curve. This depends on the number of `NetworkToken` in the reserve.
- The `Market` supports a payment layer for queries against the underlying data market. Query payments must be performed via `Market` before backends will accept queries.
  - `T_util` units of `MarketToken` are minted for the owner of a listing when that listing is queried. The backend is responsible for reporting queried listings.

  - `Market.update_listings_accessed(element_id)` can only be called by a backend for the market.
- Token holders in `Market` belong to one of two classes, data owner and investor.
  - `Market.convert_to_investor()` converts a data owner to an investor.
  - `Market.get_total_number_investor_tokens()` returns the total number of MarketTokens held by investors. This method will be used by Market.divest() and Market.get_current_investor_price()
  - `Market.set_access_cost(listing)`: Callable by the owner of a listing to set price for accessing the listing.
  - `Market.get_access_cost(listing)`: Getter to view cost.

#### Listings  [v0.2]

A market holds a set of listings. Each listing corresponds to an element of the
`Market` which is held off-chain in some (possibly multiple) `Backend` systems.
Newcomers to the market can call `Market.apply()` to apply to have their
listing added to the market. A listing consists of an off-chain datapoint (or
datapoints) and an on-chain listing structure. We reproduce the fields of the
on-chain listing structure below.

```
struct Listing {
  bytes32 dataHash; // Hash of the off-chain data-point this listing corresponds to
  uint applicationExpiry; // Expiration date of apply stage
  bool listed; // a 'listing' if true
  address owner; // owns the listing
  uint supply; // Number of tokens in the listing (both deposited and minted).
  uint challenge; // corresponts to a poll id in Voting if present
  string data; // A pointer to the actual data this listing represents (or possibly some small data primitive)
  uint minted; // Number of Market tokens that have been minted for this listing.
}
```
![alt text][maker_flow]

[maker_flow]: Maker_Flow.png "Listing flow through data market" 

##### Applying

Applying is the process by which a new listing is added to a data market. To
apply, a market participant computes the hash of their off-chain data and
proposes the addition of their data to the market by invoking `Market.apply()`:

```
function apply(bytes32 listingHash, uint amount, string data) external
```

All applications trigger a vote on the new listing by appropriate market
stakeholders (either all token-holders or the market council). If a listing
vote is cleared, it is said to be listed. Note that application is a *minting*
event whereby new `MarketTokens` are created. More detail on this can be found
in the section on minting.

##### Challenging

Challenging is the process by which a listing in a data market can be challenged and potentially removed.

#### Market Token [v0.2]

`MarketToken` is a mintable and burnable ERC20 token. The `MarketToken` is tied to a particular `Market` and is created when the `Market` is created. Note the contrast with token curated registries, which don't hold a mechanism for minting and burning their associated token.

- Minting: Minting happens in one of three ways.
  - Minting happens when new listings are added to the market. These listings have to pass through the validation process and be whitelisted before minting occurs.
    - `listedReward` is the number of new `MarketTokens` that are created upon whitelisting.
  - Minting happens when an investor enters the market through calling `Market.invest()`. This rate is set by the algorithmic price curve.
  - Minting happens when the backend reports that a listing has been queried. This results in creation of `T_util` new tokens which are awarded to listing owner (which may be the market itself).
- Burning: Burning happens in one of two ways. 
  - If a listing is removed from the market, its associated tokens should be burned. This happens when the listing owner removes the listing or when a successful challenge forces removal of the listing.
  - If an investor class token holder divests from the market, their divested tokens are burned. The origin of the tokens being burned does not matter.

#### Voting [IN-PROGRESS]

Major decisions in the market are made by token holder vote. These decisions
include which new listings should be added to the `Market`, which challenged
listings should be removed, and more. Market creators have multiple possible
voting options.

##### All Token Holder [v0.1]
At present, decisions are made by vote of all token holders.

##### Council Member Vote [v0.3]
In near future, a threshold `T_council` will be imposed, and only token holders
who hold more than `T_council` units of `MarketToken`will be allowed to vote.
Market participants who hold more than `T_council` units of `MarketToken` are
referred to as council members. Non-council members will not be allowed to vote
on market actions in this scheme.

#### Investor and Owner Class [v0.2]

The `Market` will have two classes of token holder, investors and data owners.
Note the contrast with a `Registry` which tracks only listings and challenges, and
doesn't track investors. A new data structure `mapping` will have to be added that tracks
the class of each token holder in the data market. In addition, new token
holders will have to be entered into this `mapping`. Methods that will interact
with `mapping`:

- `Market.invest()` will add a new investor class member (if not already added)
- `Market.divest()` will check if given member is investor class and remvoe them if so
- `Market.apply()` will add the given member to the data owner class (A data owner must be listed). (This method corresponds to `Registry.apply()`)
- `Market.exit()` will remove a data owner if they are not present any further listings. (This method corresponds to `Registry.exit()`).

#### Market Reserve [v0.2]
Each Data market should hold a reserve of [`Network Token`](#network-token). Here's a brief summary of methods that interact with reserve

- `Market.invest()` adds investor Network Token to reserve and mints and returns Market Token to investor. Pricing dictated by price curve.
- `Market.divest(num_tokens)` allows investor class token holders to burn Market Token and withdraw Network Token from the reserve.
  - `divest()` first checks that its caller is an investor class token holder. If so, it computes the fractional ownership this investor has (`num_tokens/total_num_investor_tokens`). For example, if `num_tokens=5` and `total_num_investor_tokens=100`, this would be 5% fractional ownership. Then `num_tokens` market tokens are burned. Then the fractional part of the reserve belonging to this investor is transferred to the investor. For example, in the case above, 5% of the reserve would be transferred to the investor's address.

#### Algorithmic Price Curve [v0.2]
The price curve dictates the conversion rate between `NetworkToken` and `MarketToken` for new investors.

![alt text][algorithmic_price_curve]

[algorithmic_price_curve]: Algorithmic_Price_Curve.png "Protocol Flowchart"

Methods that interact with the price curve


- `Market.get_current_investment_price()` reports the current `NetworkToken`/`MarketToken` conversion rate. Mathematically, the first version will be a linear function. That is, `Market.get_current_investment_price() = base_conversion_rate + conversion_slope * Market.get_reserve_size()` where `base_conversion_rate` and `conversion_slope` are parameters defined by the market creator. 
- `Market.get_reserve_size()` returns the size of current market reserve in `NetworkToken`
- `Market.invest()` consults `Market.get_current_investment_price()` to perform the exchange.

Note that the linear form of the price curve above is not necessarily set in stone. It's likely that future iterations will allow users to choose alternate forms of the price curve.

#### Queries [v0.3]

The data in the market can be queried by users. Queries must be paid for up front and must be written in an allowable query language.

#### Query Pricing [v0.3]
The `Market` controls the payment layer for queries. Users who wish to query
the data listed in a data market must first make a payment to `Market`. Any
`Backend` associated with `Market` will check that payments have gone through
before allowing for queries.

- Listing owners can set an access cost for their listing (in network token). For listings which are owned by the market itself, a council vote is required to update this access cost.
  - `Market.set_access_cost(listing)`: Callable by listing owner to set price.
  - `Market.get_access_cost(listing)`: Getter to view cost.
- To submit a query, the querier sends a [query file](#query-language) to `Backend`. A `Backend` can set its asking price to run computation for this query. Each listing will access some specified subset of listings in market.
  - `Backend::GET_COMPUTE_COST(QUERY_FILE)`: A call to the `Backend` via REST to get the cost for running this query.
  - `Market.set_query_compute_cost(query_i, backend_j, cost)`: `query_i` is one of supported queries. `backend_j` is some approved backend. `cost` is in network token.
  - `Market.get_query_compute_cost(query_i)`: Returns cheapest cost available (TODO: More refined scheme?)
  - `Market.update_listings_accessed(query_i)`: Reports the listings which the given query will access. (TODO: How does this change as new listings are added?)

- Each query causes some loss in data privacy. A charge is leveled to account for this cost. This charge may be adaptive and depend on history of past queries by given user. This loss in price is governed by the [epsilon price curve](EpsilonPriceCurve.md).
  - `Market.get_privacy_cost(query_i)`: Cost will depend on user invoking and past queries they've run.
- The total cost for running a query is the sum of all these terms. Here's some python-esque pseudo-code
```
def get_total_cost():
  cost = 0
  query = Query to run
  listings_accessed = Market.listings_accessed(query)
  # Data access cost
  for listing in listings_accessed:
    cost += Market.get_access_cost(listing)
  # Query compute cost (assume one backend)
  cost += Market.get_query_compute_cost(query)
  # Privacy cost
  cost += Market.get_privacy_cost(query)
```

##### Epsilon Privacy Curve [v0.3]

![alt text][epsilon_price_curve]

[epsilon_price_curve]: epsilon_privacy_curve.png "Epsilon Price Curve"

The Epsilon price-curve is the tool used to price for the privacy lost in a
given query. Here, epsilon is a technical parameter, adapted from the
differential privacy literature, which is a measure of the information loss
tied to a particular query. Each query has an associated epsilon.

- `Market.get_current_privacy_price(user)` returns the current price for purchasing additional privacy budget from the epsilon price curve. This depends on the current privacy epsilon used by the provided user.
- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

##### Query Rake [v0.3]

For [query payments](#query-pricing) that come in, a portion of the query payment (the "rake") is paid into the reserve, a portion is paid to listing owners, and a portion is paid to the `Backend`. 

- The owner of a listing is paid the access cost they set with `Market.set_access_cost(listing)`
- The `Backend` system is paid the compute cost they set with `Market.set_query_compute_cost(query)`
- The reserve is paid the privacy cost `Market.get_privacy_cost(query)`

TODO: This scheme isn't finalized yet; will likely change.

#### Authorizing a Backend [v0.3]
Each data market will maintain a list of authorized `Backend` systems. A full
vote of the council (#28) will be needed to add, remove, or authorize `Backend` systems.

- `Market.get_backend_system()`: Returns list of authorized backend systems for the market
- `Market.add_backend_system(id)`: This must be authorized by a vote of the council.
- `Market.remove_backend_system(id)`: This must be authorized by a vote of the council

## Off Chain Systems [v0.2, v0.3]

The off-chain portions of the Computable protocol are responsible for storing,
querying and computing upon data. All off-chain components are called `Backend`
systems, but there are multiple types of backends. A trusted `Backend` is
operated by an entity that is trusted by the creator of a particular data
market. The operator of this trusted `Backend` is authorized by the data
market's council with this authority. By contrast, an untrusted `Backend` is
not trusted by the data market council and is *not* authorized to view
unencrypted data on the network. The untrusted `Backend`operator must use more
advanced techniques such as cryptography or trusted hardware enclaves to
perform computation since they may never see unencrypted data.

Note that the implementation of a `Backend` is not specified by this document.
A `Backend` is any system that responds to the API endpoints defined in this
section. Notably, this means a `Backend` may be closed source or proprietary.
In addition, a `Backend` may have arbitrary hardware backing it. It could run
on a single laptop or could be a cluster of servers on the cloud. It could even
use novel proprietary hardware such as GPUs, TPUs or ASICs to power needed
workloads. These choices are left to the operator of the `Backend`.

### Backend Specification [v0.3]
A `Backend` is a system that is responsible for storing data off-chain. Any `Market` contains a list of authorized `Backend`s which hold the raw data associated with the `Market`.

Broadly speaking, `Backend`s are either trusted or untrusted. A trusted `Backend` is allowed to view the unencrypted data that belongs to a `Market`. On the other hand, an untrusted `Backend` is never allowed to view unencrypted data belonging to the data market.

A `Backend` is responsible for serving queries against a given `Market`. Each query is sent as a file in an acceptable [query language](QueryLanguage.md). 

#### REST API [v0.3]

The `Backend` is responsible for serving a number of endpoints. These endpoints are specified below. The syntax `Backend::ENDPOINT(input)` is used to specify that the `Backend` supports an `ENDPOINt` which expects `input`.

- `Backend::AUTHENTICATE(user_credentials)`: Authenticates the given user. Returns an authorization token.
- `Backend::GET_SCHEMA(market)`: Returns the schema for the given market.
- `Backend::RUN_QUERY(auth_token, markets, query)`: Runs the given query against the given markets. The query is sent in a query file.
- `Backend::ADD_DATA(auth_token, market, data)`: Adds the given data point to the `Backend` along with necessary authorization.
- `Backend::REMOVE_DATA(auth_token, market, data)`: Removes the given data point from the `Backend`.
- `Backend::MARKETS_SUPPORTED()`: Returns the list of `Market`s which `Backend` supports.
- `Backend::GET_COMPUTE_COST(query)`: A call to the `Backend` via REST to get the cost for running specified query.
- `Backend::GET_EPSILON(query)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

Let's dive into these methods in more detail.

##### AUTHENTICATE
Calling `Backend::AUTHENTICATE(user_credentials)` will authenticate the given
user with the particular Backend at hand. An authenticated user will be able to
submit queries directly to the Backend system without needing to constantly
communicate with the blockchain itself. Note however, that the authentication
process *will* check against the blockchain to see if a particular user is
authorized to access the data on this Backend system. This means that the
`user_credentials` will likely include a signature using the user's private
key.

Note in addition that the implementation of the specific authentication system
is not specified in this document. Different backend creators may choose to
create separate implementations. In addition, note that each Backend system is
responsible for implementing its own authentication layer. However, since a
given Backend may serve many data markets, an authenticated user may be able to
easily query against multiple markets or run multi-market queries on a given
Backend system.


##### GET_SCHEMA

`Backend::GET_SCHEMA(market)` returns the schema associated with a particular
data market. Note that the schema is associated closely with an individual
market. In a future iteration of the design, the schema may be an on-chain
entity.

##### RUN_QUERY
Calling `Backend::RUN_QUERY(auth_token, markets, query)` will execute the provided
query. Note that an `auth_token` is required since the user running the query
must be authorized to do so. Note in addition that `query` may be run against a
set of `markets` (where `markets` is a list of strings specifying the names of
different markets). For such queries, `markets` must be a subset of
`Backend::MARKETS_SUPPORTED()`.

We don't specify in this document the structure of the query engine.
Conceivably, queries could include SQL, machine learning scripts, ETL
transformations and more. A future iteration of this document may specify the
precise forms of query strings accepted by this API.

##### ADD_DATA

The call `Backend::ADD_DATA(auth_token, market, data)` adds the specified
`data` to the specified `market`. Here `data` is a list of datapoints. Each
datapoint is a bytestring. The hash of this bytestring must be listed within
the on-chain data market contract as approved in order for `ADD_DATA` to
succeed.

##### REMOVE_DATA
The call `Backend::REMOVE_DATA(auth_token, market, data)` removes the specified
`data` from the specified market. As before, `data` is alist of datapoints,
where each datapoint is a bytestring. `data` must have previously been added in
a call to `ADD_DATA`. In addition, the hash of this bytestring must appear as
the hash of a listing that has been *removed* from the on-chain data market.
(Note that the on-chain ledger is maintains the listings for removed
data-points on-chain even after they are no longer formally listed).

##### MARKETS_SUPPORTED
The call to `Backend::MARKETS_SUPPORTED()` returns a list of the markets which
are supported on this Backend. Note that no authentication is needed for this
call.

##### GET_COMPUTE_COST

The `Backend::GET_COMPUTE_COST(query)` API call provides a Backend system's
estimated cost to run the query. Note that the method in which
a particular Backend system estimates these costs is not specified in this
document. Different Backend systems may have different cost estimation methods.
For example, a Backend could use current price estimates on cloud providers to
set cost, or perhaps its own internal statistics.

##### GET_EPSILON
The call to `Backend::GET_EPSILON(query)` computes the differential privacy
parameter epsilon that is associated with this given query. Note that this call
may sometimes fail, when it is not possible to compute epsilon for the query at
hand.


#### Query Language [v0.3]

![alt text][query_flow]

[query_flow]: QueryFlow.png "Query Flow"

Queries to a `Backend` node must be in a recognized query language. These queries are sent to the `Backend` system within query files.
- SQL: A subset of SQL are allowed.
- Python: Queries are allowed the be phrased in a restricted subset of python. This subset does not allow for network or filesystem access. In addition, the data tables are pre-loaded.

![alt text][multi_market_join]

[multi_market_join]: Multi_Market_Join.png "Multi Market Join"

#### Data Utilization [v0.3]

The market is responsible for maintaining record of which queries have accessed which datapoints. The backend system will report datapoints accessed by a given query to the market.

- `Market.update_listings_accessed(query_i)`: Called by backend system after running a query. This information is stored on-chain. For reasons of gas, this may just be a simple count; each listing may maintain a simple count field which is incremented for each additional query that accesses it. (See also discussion in #32 around pricing)

## Case Studies

Thus far, we've discussed the Computable protocol in the abstract. In this section, we walk through a few case studies of different types of data markets that can be constructed using the Computable protocol.

### Censorship Resistant Data Markets

It is possible to build data markets that are resistant to censorship efforts.

![alt text][censorship_resistant_data_markets]

[censorship_resistant_data_markets]: Censorship_Resistant_Data_Market.png "Censorship Resistant Data Markets"


