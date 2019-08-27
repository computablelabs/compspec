# Voting

Major decisions in the data market are made by token
holder vote.  These decisions include which new
listing candidates should be confirmed as listings, which
challenged listings should be removed, what changes
should be made to the `Parameterizer` parameters, and
what the datatrust should be. In this chapter, we'll
introduce you to the fundamentals of the voting
system and describe the basics of on-chain Computable
governance.

## Candidates, Candidates, Candidates

All voting is done on "candidates." Think of a
candidate as governance question brought up for
referendum. There are a number of different types of
candidates:

- Listing candidates: A proposal to add a new listing
  to the data market.
- Challenge candidate: A proposal to remove an existing
  listing from the data market.
- Reparameterization candidate: A proposal to change
  the parameters of the data market.
- Datatrust candidate: A proposal to change the
  datatrust used for the market.

On-chain, a candidate is represented by a `Candidate`
struct:

```
struct Candidate:
  kind: uint256 # one of [1,2,3,4] representing an
application, challenge, reparam or registration
respectively
  owner: address
  stake: wei_value
  vote_by: timestamp
  yea: uint256
  nay: uint256
``` 

Let's quickly review the fields of this struct. The
`owner` is the Ethereum address of the candidate
proposer. The `stake` is the amount of `MarketToken`
that must be placed as stake to vote for/against this
candidate. (We'll say more about this shortly).
`vote_by` is how long the the voting poll will be open
for this candidate. And `yea` and `nay` count the
number of votes in favor and opposing this candidate.

## Voting

The voting system is quite simple. To place a vote, a
stakeholder simply locks up `Candidate.stake` of its
`MarketToken`.

```
@public
def vote(hash: bytes32, option: uint256):
  """
  @notice Cast a vote for a given candidate
  @dev User mush have approved market token to spend on
their behalf
  @param hash The candidate identifier
  @param option Yea (1) or Nay (!1)
  """
  assert self.candidates[hash].owner != ZERO_ADDRESS
  assert self.candidates[hash].vote_by >
block.timestamp
  stake: wei_value = self.candidates[hash].stake
  self.market_token.transferFrom(msg.sender, self,
stake)
  self.stakes[msg.sender][hash] += stake
  if option == 1:
    self.candidates[hash].yea += 1
  else:
    self.candidates[hash].nay += 1
  log.Voted(hash, msg.sender)
```

A stakeholder may vote as many times as they wish, at
the cost of locking up more stake. Note that this makes
a data market an explicit plutocracy: larger
stakeholders in the data market explicitly have greater
voting power. While the downsides of plutocratic
systems are well known, it's worth remembering that
each data market is a relatively local system. It's not
unfair that large stakeholders have more say in how its
run (just as owners of a business have more say in its
operations than third parties).

The vote passes if the proportion of `yea` votes is
greater than the `plurality` parameter.

```
@public
@constant
def didPass(hash: bytes32, plurality: uint256) -> bool:
  """
  @notice Return a bool indicating whether a given
candidate recieved enough votes to exceed the plurality
  @dev The poll must be closed. Also we cover the
corner case that no one voted.
  @return bool
  """
  assert self.candidates[hash].owner != ZERO_ADDRESS
  assert self.candidates[hash].vote_by <
block.timestamp
  yea: uint256 = self.candidates[hash].yea
  total: uint256 = yea + self.candidates[hash].nay
  # edge case that no one voted
  if total == 0:
    # theoretically a market could have a 0 plurality
    return plurality == 0
  else:
    return ((yea * 100) / total) >= plurality
```

This simple voting system has the advantage of being
easy to understand. However, the trade-off is that this
simplicity allows for some sophisticated vote
manipulation techniques to be feasible. We discuss such
attacks in greater detail later.
