The `Market` controls the payment layer for queries. Users who wish to query
the data listed in a data market must first make a payment to `Market`. Any
`Backend` associated with `Market` will check that payments have gone through
before allowing for queries.

- Listing owners can set an access cost for their listing (in network token). For listings which are owned by the market itself, a council vote is required to update this access cost.
  - `Market.set_access_cost(listing)`: Callable by listing owner to set price.
  - `Market.get_access_cost(listing)`: Getter to view cost.
- To submit a query, the querier sends a [query file](QueryLanguage.md) to `Backend`. A `Backend` can set its asking price to run computation for this query. Each listing will access some specified subset of listings in market.
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
