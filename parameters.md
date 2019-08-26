# Market Parameters

- [On-chain smart contracts](#on-chain-components):
  - [Market Parameters](#market-parameters): The `Market` is governed by a set of a parameters dictated within the `Parameterizer`.
      - [Reparameterization](#reparameterization): The parameters that govern the `Market` can be modified with a council vote.

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

