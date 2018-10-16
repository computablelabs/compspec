The `MarketFactory` is responsible for creaking new data markets and will store a list of available data markets.

- `MarketFactory.create_data_market()`: Constructs and launches a new data market. This is the only public way to create a new data market. There are a number of arguments needed in this constructor.
Each data market has an associated token with it. `create_data_market()` should pass in necessary information to initialize this token. It might make sense to pass in a list of `[address_1: allocation_1,...,address_n:allocation_n]` of initial token allocations to `create_data_market()`. The Market token would be initialized with this initial spread of market token.
- `T_council`: Council membership threshold fraction #28
- `T_util`: Number of tokens minted when a listing is queried #31
- `T_submit`: Number of tokens minted when a new listing is listed #31
- `MarketFactory.get_list_of_data_markets()`: Returns a list of available data markets.
