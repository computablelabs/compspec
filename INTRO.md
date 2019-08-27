---
title: The Computable Protocol 
type: docs
---

# The Computable Protocol 

## Introduction 

The Computable protocol creates decentralized data
markets.  The global Computable network is made up of
many individual markets. Each market conceptually holds
a single collection of data and is created and
controlled by the owners of this data. These owners
could correspond to existing organizations, or could be
a decentralized set of interested parties. The
coordination and access control for these individual
market instances is coordinated by a set of [smart
contracts](https://en.wikipedia.org/wiki/Smart_contract).
Each market allows for a set of associated financial
operations. These operations allow interested "patrons"
to support a particular Market with funds, "makers" to
contribute data in return for partial ownership in the
market, and "buyers" to purchase data from the Market.
To facilitate these transactions, each market has a
unique associated `MarketToken`.

Everything described above is implemented in a set of
smart contracts which currently live on the
[Ethereum](https://en.wikipedia.org/wiki/Ethereum)
blockchain. The data itself doesn't live on the smart
contracts. Datasets can be very large (gigabytes,
terabytes, petabytes, exabytes or more) and smart
contract systems have limited "on-chain" storage.
Consequently, it would be infeasible to store such
large collections of data on Ethereum. For this reason,
data lives "off-chain" in `Datatrusts`. A `Datatrust`
is a software system that is responsible for storing
data and coordinating with on-chain permissions layers.
Note that many possible `Datatrust` implementations are
possible by different vendors or groups, so long as
each implementation responds to the API specified
within this specification. 

This document is a living, versioned specification. As
understanding of the core aspects of the Computable
protocol grows, this document will be updated
accordingly.

![Protocol Flowchart](market_transactions.png)
