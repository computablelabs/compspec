A `Backend` is a system that is responsible for storing data off-chain. Any `Market` contains a list of authorized `Backend`s which hold the raw data associated with the `Market`.

Broadly speaking, `Backend`s are either trusted or untrusted. A trusted `Backend` is allowed to view the unencrypted data that belongs to a `Market`. On the other hand, an untrusted `Backend` is never allowed to view unencrypted data belonging to the data market.

A `Backend` is responsible for serving queries against a given `Market`. Each query is sent as a file in an acceptable [query language](QueryLanguage.md). The `Backend` is responsible for serving a number of endpoints. These endpoints are specified below. The syntax `Backend::ENDPOINT(input)` is used to specify that the `Backend` supports an `ENDPOINt` which expects `input`.

- `Backend::AUTHENTICATE(user_credentials)`: Authenticates the given user. Returns an authorization token.
- `Backend::GET_SCHEMA(market)`: Returns the schema for the given market.
- `Backend::RUN_QUERY(query_file)`: Runs the given query. The query is sent in a query file.
- `Backend::ADD_DATAPOINT(auth_token, datapoint)`: Adds the given data point to the `Backend` along with necessary authorization.
- `Backend::REMOVE_DATAPOINT(auth_token, datapoint)`: Removes the given data point from the `Backend`.
- `Backend::MARKETS_SUPPORTED()`: Returns the list of `Market`s which this backend supports.
