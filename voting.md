# Voting

As we've mentioned previously, `MarketToken`s are
analogous to company shares. Just as company
shareholders can vote on some company decisions,
`MarketToken` stakeholders are allowed to vote on some
data market governance questions.  In fact, all major
decisions in the data market are made by token holder
vote.  These decisions include which new listing
candidates (a "listing candidate" is a chunk of data
which has not yet been confirmed as a listing in the
market) should be confirmed as listings, which
challenged listings should be removed from the market
(we'll say more about challenges later), what changes
should be made to the `Parameterizer` parameters (which
govern the market's behavior; more on this later), and
who the datatrust operator should be. In this chapter,
we'll introduce you to the fundamentals of the voting
system and describe the basics of on-chain Computable
governance.

## Candidates, Candidates, Candidates

All voting is done on "candidates." Think of a
candidate as governance question brought up for
referendum. There are a number of different types of
candidates:

### Listing candidates

A proposal to add a new listing to the data market is a
"listing candidate." A maker (a data contributor) who
has gathered some interesting data, creates a listing
candidate to propose that their data should be added to
the data market. If this candidate is accepted, the
maker is rewarded with `listing_reward`, a tranche of
newly minted `MarketToken`. 

Here, the vote serves as a gating mechanism to prevent
fraudulent data from entering the market. It also
allows for some quality control checks on data to be
performed by any interested parties.

### Challenge candidate

A proposal to remove an existing listing from the data
market. Sometimes, the listing process will allow a bad
listing (with poor data perhaps) to slip through. In
this case, we need a mechanism to allow for clean-up
and removal of this data. The challenge mechanism
allows for this sort of cleanup. An interested party
can challenge a listing they believe to be fraudulent.
If the vote succeeds, this listing is removed from the
market.

To prevent nuisance challenges, creating a challenge
requires locking up some `MarketToken`. The amount to
be locked up is governed by the `stake` parameter set
in the `Parameterizer`. If the challenge is rejected
(judged by stakeholders to be frivolous), then the
listing owner is rewarded for the trouble they faced.
For this reason, challenges will likely be relatively
uncommon since there's a risk of losing funds.

As a second complication, you might ask, what happens
to the `MarketToken` that was minted in the case of a
successful challenge that removes the associated
listing? The answer is absolutely nothing. If you
succeeded in pulling a fast one on the data market, you
are allowed to walk with the funds. This choice was
made to keep the contract complexity to manageable
levels, since having "lockups" to prevent this
situation would add quite a bit of extra code. 

### Reparameterization candidate

A proposal to change the parameters of the data market. We'll say more about the parameters that can be altered in a future chapter. 

### Datatrust candidate
A proposal to change the datatrust used for the market.
If a datatrust operator is misbehaving, this mechanism
allows for the datatrust to be replaced if necessary.
However, it's worth nothing that the datatrust has
considerable power in this version of the Computable
protocol, so this facility is something of a "nuclear
option" only to be used if other measures aren't
panning out.

## The voting code

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
proposer. As a quick note, the Computable protocol
doesn't know anything about humans. The only entities
that it knows about are Ethereum addresses. What is an
address you might ask? It's simply a 64 character hex
string that uniquely identifies some entity on
Ethereum. This entity could be a human, a smart
contract, a consortium or anything. All the
participants in the Computable protocol we've talked
about (makers, stakeholders, datatrust operators) are
all Ethereum addresses. Which means that participants
in the Computable protocol aren't necessarily humans,
although the could be of course.

Returning to the discussion, the `stake` is the amount
of `MarketToken` that must be placed as stake to vote
for/against this candidate. (We'll say more about this
shortly). `vote_by` is how long the the voting poll
will be open for this candidate. And `yea` and `nay`
count the number of votes in favor and opposing this
candidate.

## Voting

The voting system is quite simple. To place a vote, a
stakeholder simply locks up `Candidate.stake` of
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

## Last Thoughts

Now that you understand how voting works, let's dive
into the most important thing to be governed by votes,
the listings themselves. You'll learn more in the next
chapter.

<div style="text-align: right"> <a href="../../docs/listings">Next Chapter</a> </div>
