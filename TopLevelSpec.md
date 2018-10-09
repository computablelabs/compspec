The current specification for the protocol is spread across many GitHub issues. This issue gathers and organizes these more focused specifications:

- `DataMarketFactory` [#29](https://github.com/computablelabs/goest/issues/29): The top level entry point to create a new market and associated token.
- `NetworkToken` [#19](https://github.com/computablelabs/goest/issues/19) The top level token for the entire network. (TODO: More description of function of this token)
- `Market` [14](https://github.com/computablelabs/goest/issues/14) The top level contract for a given data market.
  - `MarketToken` [#12](https://github.com/computablelabs/goest/issues/12): A mintable and burnable token. Each `Market` has its own `MarketToken`
    - Minting and Burning mechanics [#31](https://github.com/computablelabs/goest/issues/31): Market tokens are minted when either new data is added, existing data is queried, or new investment is added to reserve. Market tokens are burned when data is removed or investment is withdrawn.
  - Voting: Critical decisions within a market are performed by vote of interested stake holders. These include validation of new data, challenges to fraudulent data and changes to market structure.
    - All token holder vote [#13](https://github.com/computablelabs/goest/issues/13): At present all holders of `MarketToken` vote on decisions.
    - Council member vote [#28](https://github.com/computablelabs/goest/issues/28): In v3, an ownership threshold `T_council` will be imposed for franchise. The threshold will be set upon construction.
  - The Reserve [#20](https://github.com/computablelabs/goest/issues/20): The reserve is the "bank account" associated with the market. 
    - The algorithmic price curve [#21](https://github.com/computablelabs/goest/issues/21): Controls the price at which new investors may invest in market. Investor funds are deposited in reserve and new market token is minted accordingly.
    - Investor and data owner class tokens [#22](https://github.com/computablelabs/goest/issues/22): Holders of market token are investor class or data owner class. Investor class tokens can't own any listings in the market, but have right to withdraw funds from reserve by burning their tokens. Data owner class tokens can own listings in market, but can't withdraw funds from reserve. (TODO: Can one address hold some data owner class and some investor class tokens? Potential attacks?)
  - Queries: Each data market supports some set of queries against the data in this market. Queries are run on backend systems. (TODO: How are queries specified?)
    - Query Pricing [#32](https://github.com/computablelabs/goest/issues/32): Users have to pay to run queries. This pricing structure has to reward the various stakeholders including listing owners (data), backend system owners (compute), and the market itself (investors)
    - Query Rake [#33](https://github.com/computablelabs/goest/issues/33): What fraction of the payment goes to each stake holder?
    - Data utilization records [#35](https://github.com/computablelabs/goest/issues/35): The market maintains track of how many times each listing has been requested by different queries.
  - Off-chain data [#30](https://github.com/computablelabs/goest/issues/30): The data listed in the data market is held off-chain in a backend system. A council vote is used to set authorized backend systems for this market.
- Backend Systems: Backend systems are responsible for securely storing data off-chain and allowing authorized users to query this data. https://github.com/computablelabs/crunky
  - [REST API](https://github.com/computablelabs/crunky/issues/1): The backend system must respond to a defined set of REST API commands to perform actions such as authentication, data addition and removal, and query handling 
  - [Query specification](https://github.com/computablelabs/crunky/issues/2): The backend system must accept a structured set of queries (TODO: More details and work needed here)

  This specification is incomplete. As clarity is gained on the various TODOs and other holes in the specification, this top level spec will be edited.
