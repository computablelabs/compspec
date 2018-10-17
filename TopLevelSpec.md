# The Computable Protocol

The Computable protocol creates a decentralized data market. Conceptually, the
`Computable` network contains many `Market`s. Each `Market` is a data market
that controls access to a single dataset. The dynamics of a given `Market` are
governed by its associated `MarketToken`. Note that different `Market`s have
different `MarketTokens`. The global `Computable` network uses a
`NetworkToken`.

![alt text][protocol_flowchart]

[protocol_flowchart]: Protocol_Flowchart.png "Protocol Flowchart"


## Top Level Specification

This section provides a high level roadmap of the full protocol with links to more detailed specifications in subsequent sections.

- [`MarketFactory`](#market-factory): The top level entry point to create a new market and associated token.
- [`NetworkToken`](#network-token) The top level token for the entire network.
- [`Market`](#market) The top level contract for a given data market.
  - [`MarketToken`](#market-token): A mintable and burnable token. Each `Market` has its own `MarketToken`
    - Minting and Burning mechanics: Market tokens are minted when either new data is added, existing data is queried, or new investment is added to reserve. Market tokens are burned when data is removed or investment is withdrawn.
  - Voting: Critical decisions within a market are performed by vote of interested stake holders. These include validation of new data, challenges to fraudulent data and changes to market structure.
    - [All token holder vote](#all-token-holder): At present all holders of `MarketToken` vote on decisions.
    - [Council member vote](#council-member-vote): an ownership threshold `T_council` is imposed for franchise. The threshold will be set upon construction.
  - [Market Reserve](#market-reserve): The reserve is the "bank account" associated with the market. 
    - The [algorithmic price curve](#algorithmic-price-curve): Controls the price at which new investors may invest in market. Investor funds are deposited in reserve and new market token is minted accordingly.
    - [Investor and data owner class tokens](#investor-and-owner-class): Holders of market token are investor class or data owner class. Investor class tokens can't own any listings in the market, but have right to withdraw funds from reserve by burning their tokens. Data owner class tokens can own listings in market, but can't withdraw funds from reserve. (TODO: Can one address hold some data owner class and some investor class tokens? Potential attacks?)
  - Queries: Each `Market` supports queries against the data in this market. Queries are run on a `Backend` tied to the market and can be specified in a supported [query language](#query-language)
    - [Query Pricing](#query-pricing): Users have to pay to run queries. This pricing structure has to reward the various stakeholders including listing owners (data), backend system owners (compute), and the market itself (investors)
    - [Query Rake](#query-rake): What fraction of the payment goes to each stake holder?
    - [Data utilization](#data-utilization): The market maintains track of how many times each listing has been requested by different queries.
  - Off-chain data [#30](https://github.com/computablelabs/goest/issues/30): The data listed in the data market is held off-chain in a backend system. A council vote is used to set authorized backend systems for this market.
- [Backend Systems](#backend-specification): Backend systems are responsible for securely storing data off-chain and allowing authorized users to query this data. https://github.com/computablelabs/crunky
  - [REST API](https://github.com/computablelabs/crunky/issues/1): The backend system must respond to a defined set of REST API commands to perform actions such as authentication, data addition and removal, and query handling 
  - [Query specification](https://github.com/computablelabs/crunky/issues/2): The backend system must accept a structured set of queries (TODO: More details and work needed here)

## Market Factory
The `MarketFactory` is responsible for creaking new data markets and will store a list of available data markets.

- `MarketFactory.create_data_market()`: Constructs and launches a new data market. This is the only public way to create a new data market. There are a number of arguments needed in this constructor.
Each data market has an associated token with it. `create_data_market()` should pass in necessary information to initialize this token. It might make sense to pass in a list of `[address_1: allocation_1,...,address_n:allocation_n]` of initial token allocations to `create_data_market()`. The Market token would be initialized with this initial spread of market token.
- `T_council`: Council membership threshold fraction #28
- `T_util`: Number of tokens minted when a listing is queried #31
- `T_submit`: Number of tokens minted when a new listing is listed #31
- `MarketFactory.get_list_of_data_markets()`: Returns a list of available data markets.

## Network Token
The `NetworkToken` is the central token that powers the Computable network. It
is used by the [MarketFactory](MarketFactory.md) to perform operations and is
used to pay for queries executed by a `Backend`. The `NetworkToken` is implemented
by a `StandardToken` (ERC) for now. 

## Market

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

	- `Market.report_accessed(element_id)` can only be called by a backend for the market.
- Token holders in `Market` belong to one of two classes, data owner and investor.
	- `Market.convert_to_investor()` converts a data owner to an investor.
  - `Market.get_total_number_investor_tokens()` returns the total number of MarketTokens held by investors. This method will be used by Market.divest() and Market.get_current_investor_price()
  - `Market.set_access_cost(listing)`: Callable by the owner of a listing to set price for accessing the listing.
  - `Market.get_access_cost(listing)`: Getter to view cost.

### Basic Structure of Market

A market holds a set of listings. Each listing corresponds to an element of the `Market` which is held off-chain in some (possibly multiple) `Backend`. Newcomers to the market can call `Market.apply()` to apply to have their listing added to the market.

### Market Token

`MarketToken` is a mintable and burnable ERC20 token. The `MarketToken` is tied to a particular `Market` and is created when the `Market` is created. Note the contrast with token curated registries, which don't hold a mechanism for minting and burning their associated token.

- Minting: Minting happens in one of three ways.
  - Minting happens when new listings are added to the market. These listings have to pass through the validation process and be whitelisted before minting occurs.
    - `listedReward` is the number of new `MarketTokens` that are created upon whitelisting.
  - Minting happens when an investor enters the market through calling `Market.invest()`. This rate is set by the algorithmic price curve.
  - Minting happens when the backend reports that a listing has been queried. This results in creation of `T_util` new tokens which are awarded to listing owner (which may be the market itself).
- Burning: Burning happens in one of two ways. 
  - If a listing is removed from the market, its associated tokens should be burned. This happens when the listing owner removes the listing or when a successful challenge forces removal of the listing.
  - If an investor class token holder divests from the market, their divested tokens are burned. The origin of the tokens being burned does not matter.

### Voting Structure

Decisions in the market are made by token holder vote.

#### All Token Holder
At present, decisions are made by vote of all token holders.

#### Council Member Vote
In near future, a threshold `T_council` will be imposed, and only token holders
who hold more than `T_council` units of `MarketToken`will be allowed to vote.
Market participants who hold more than `T_council` units of `MarketToken` are
referred to as council members. Non-council members will not be allowed to vote
on market actions in this scheme.

### Investor and Owner Class

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

### Market Reserve
Each Data market should hold a reserve of [`Network Token`](#network-token). Here's a brief summary of methods that interact with reserve

- `Market.invest()` adds investor Network Token to reserve and mints and returns Market Token to investor. Pricing dictated by price curve.
- `Market.divest(num_tokens)` allows investor class token holders to burn Market Token and withdraw Network Token from the reserve.
  - `divest()` first checks that its caller is an investor class token holder. If so, it computes the fractional ownership this investor has (`num_tokens/total_num_investor_tokens`). For example, if `num_tokens=5` and `total_num_investor_tokens=100`, this would be 5% fractional ownership. Then `num_tokens` market tokens are burned. Then the fractional part of the reserve belonging to this investor is transferred to the investor. For example, in the case above, 5% of the reserve would be transferred to the investor's address.

### Algorithmic Price Curve
The price curve dictates the conversion rate between Network Token and Market Token. Note this issue is a dependency for #14. Methods that interact with the price curve

- `Market.get_current_investment_price()` reports the current NetworkToken/MarketToken conversion rate. Mathematically, the first version will be a linear function. That is, `Market.get_current_investment_price() = base_conversion_rate + conversion_slope * Market.get_reserve_size()` where `base_conversion_rate` and `conversion_slope` are parameters defined by the market creator. 
- `Market.get_reserve_size()` returns the size of current market reserve in NetworkToken
- `Market.invest()` consults `Market.get_current_investment_price()` to perform the exchange.

Note that the linear form of the price curve above is not necessarily set in stone. It's likely that future iterations will allow users to choose alternate forms of the price curve.

### Query Pricing
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
  - `Market.report_listings_accessed(query_i)`: Reports the listings which the given query will access. (TODO: How does this change as new listings are added?)

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

#### Epsilon Privacy Curve

![alt text][epsilon_price_curve]

[epsilon_price_curve]: epsilon_privacy_curve.png "Epsilon Price Curve"

The Epsilon price-curve is the tool used to price for the privacy lost in a
given query. Here, epsilon is a technical parameter, adapted from the
differential privacy literature, which is a measure of the information loss
tied to a particular query. Each query has an associated epsilon.

- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

#### Query Rake

For [query payments](#query-pricing) that come in , a portion of the query payment (the "rake") is paid into the reserve, a portion is paid to listing owners, and a portion is paid to the `Backend`. 

- The owner of a listing is paid the access cost they set with `Market.set_access_cost(listing)`
- The `Backend` system is paid the compute cost they set with `Market.set_query_compute_cost(query)`
- The reserve is paid the privacy cost `Market.get_privacy_cost(query)`

TODO: This scheme isn't finalized yet. Leave comments below and will update this top level specification.

## Backend Specification
A `Backend` is a system that is responsible for storing data off-chain. Any `Market` contains a list of authorized `Backend`s which hold the raw data associated with the `Market`.

Broadly speaking, `Backend`s are either trusted or untrusted. A trusted `Backend` is allowed to view the unencrypted data that belongs to a `Market`. On the other hand, an untrusted `Backend` is never allowed to view unencrypted data belonging to the data market.

A `Backend` is responsible for serving queries against a given `Market`. Each query is sent as a file in an acceptable [query language](QueryLanguage.md). 

### REST API

The `Backend` is responsible for serving a number of endpoints. These endpoints are specified below. The syntax `Backend::ENDPOINT(input)` is used to specify that the `Backend` supports an `ENDPOINt` which expects `input`.

- `Backend::AUTHENTICATE(user_credentials)`: Authenticates the given user. Returns an authorization token.
- `Backend::GET_SCHEMA(market)`: Returns the schema for the given market.
- `Backend::RUN_QUERY(query_file)`: Runs the given query. The query is sent in a query file.
- `Backend::ADD_DATAPOINT(auth_token, datapoint)`: Adds the given data point to the `Backend` along with necessary authorization.
- `Backend::REMOVE_DATAPOINT(auth_token, datapoint)`: Removes the given data point from the `Backend`.
- `Backend::MARKETS_SUPPORTED()`: Returns the list of `Market`s which `Backend` supports.
- `Backend::GET_COMPUTE_COST(QUERY_FILE)`: A call to the `Backend` via REST to get the cost for running specified query.
- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

### Query Language
Queries to a `Backend` node must be in a recognized query language.
- SQL: A subset of SQL are allowed.
- Python: Queries are allowed the be phrased in a restricted subset of python. This subset does not allow for network or filesystem access. In addition, the data tables are pre-loaded.

### Data Utilization

The market is responsible for maintaining record of which queries have accessed which datapoints. The backend system will report datapoints accessed by a given query to the market.

- `Market.report_listings_accessed(query_i)`: Called by backend system after running a query. This information is stored on-chain. For reasons of gas, this may just be a simple count; each listing may maintain a simple count field which is incremented for each additional query that accesses it. (See also discussion in #32 around pricing)
