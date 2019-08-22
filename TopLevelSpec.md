# The Computable Protocol


## Top Level Specification

  - [On-chain smart contracts](#on-chain-components):
		- [Market Parameters](#market-parameters): The `Market` is governed by a set of a parameters dictated within the `Parameterizer`.
        - [Reparameterization](#reparameterization): The parameters that govern the `Market` can be modified with a council vote.
- [Forward Looking Research](#forward-looking-research): Features in this section are currently being researched with the goal of eventual inclusion into the core Computable protocol. However, these features are not yet formally on the roadmap for any given Computable version release.
  - [Epsilon Privacy Curve](#epsilon-privacy-curve): A curve that prices queries by the amount of privacy loss they cost to the data market owner.
- [Case Studies](#case-studies) We consider a few case studies of interesting data markets that can be constructed with the Computable protocol in this section.
  - [Censorship Resistant Data Market](#censorship-resistant-data-markets) The Computable protocol allows for the construction of data markets that are resistant to censorship efforts.



### On Chain Components

The on-chain components of the protocol control
economics and access control.  If a user wants to gain
access to a particular dataset (in a particular data
market), or if a user wants to invest in a particular
data market, they have to seek on-chain authorization.
If a user wants to pay for queries, this is also done
off-chain. The advantage of this structure is that
payments and authorization can be handled securely by
secure on-chain contracts.

#### Minting and Burning Mechanics

Burning happens in the scenarios explained below.
- If a listing is removed from the `Market`, its associated tokens are burned. This happens when the listing owner removes the listing or when a successful challenge forces removal of the listing.
- If an investor class token holder divests from the `Market`, their divested tokens are burned. The origin of the tokens being burned does not matter.



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

### Epsilon Privacy Curve

![alt text][epsilon_price_curve]

[epsilon_price_curve]: epsilon_privacy_curve.png "Epsilon Price Curve"

The Epsilon price-curve is the tool used to price for
the privacy lost in a given query. Here, epsilon is a
technical parameter, adapted from the differential
privacy literature, which is a measure of the
information loss tied to a particular query. Each query
has an associated epsilon. Here are some possible APIs
for this feature.

- `Market.get_current_privacy_price(user)` returns the current price for purchasing additional privacy budget from the epsilon price curve. This depends on the current privacy epsilon used by the provided user.
- `Backend::GET_EPSILON(QUERY_FILE)`: A call to the `Backend` via REST to get the epsilon privacy loss for running specified query.

### Differential Privacy on Queries


![alt text][query_flow]

[query_flow]: QueryFlow.png "Query Flow"

![alt text][multi_market_join]

[multi_market_join]: Multi_Market_Join.png "Multi Market Join"


### Censorship Resistant Data Markets

It is possible to build data markets that are resistant to censorship efforts.

![alt text][censorship_resistant_data_markets]

[censorship_resistant_data_markets]: Censorship_Resistant_Data_Market.png "Censorship Resistant Data Markets"

