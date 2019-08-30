# Market Token

`MarketToken` is a mintable and burnable ERC-20 token
(we'll say more on what those terms mean soon).  The
`MarketToken` is tied to a particular data market and
is created when the data market is created. What is the
purpose of `MarketToken`? Well simply put, the
`MarketToken` corresponds to "shares" in a dataset. In
everyday life, you've probably bought share in
companies. These shares give you some rights. You can
vote on some company decisions, and have rights to some
portion of the company's wealth (albeit in limited
form). The `MarketToken` is similar. It gives you
partial ownership of the dataset that is hosted in the
data market along with some governance rights. We'll
explain the function of the `MarketToken` in greater
detail in this and subsequent chapters.

One very important point to note here is that each data
market has a different `MarketToken`. Let's unpack that
statement a bit. The Computable ecosystem consists of
many different data markets. Each data market holds a
collection of data that is logically tied together in
some fashion. For example, data market "A" might hold
mapping data for self driving cars, and data market "B"
might hold biological experimental data for drug
discovery. There will then exist *separate* market
tokens for each market. That is, `MarketToken` "A" and
`MarketToken` "B". These two tokens have nothing to do
with one another! You might wonder why this is the
case? Well it's for the same reason that Google shares
and Merck shares don't have anything to do with one
another; they're separate companies that should be
valued on their own merits independently. Similarly,
different data markets have different `MarketToken`s to
allow for separate valuations.

Another interesting feature of `MarketToken` is
that tokens are dynamically minted and burned as the
data market evolves. This flexibility is needed to
accurately track the evolving value of data in a data
market.  `MarketTokens` are minted in one of a few
scenarios explained in the next section.

## Minting 

What does it mean for `MarketToken` to be minted? Very
mechanically, it means that new `MarketToken`s are
created. Like suppose that a company had 100 shares,
and they decide to make 50 new shares and sell them to
people to raise funds. There are now 150 shares. This
is a minting operation. The mechanism works very
similiarly for data markets. In the rest of this
section, we'll talk through the situations in which
`MarketToken` can be minted.

Minting happens when new data is added to the market.
The owner of the data is rewarded with some new
`MarketToken`. In a data market, a new chunk of data
has a special name, a "listing." To prevent rampant
inflation, the addition of a new listing to the market
requires the approval of existing `MarketToken` holders
by a vote. You can think of this as a sort of
"shareholder vote." More precisely, this is called a "stakeholder vote." Just as the owners of Google stock are Google shareholders, the owners of `MarketToken` for a particular market are "stakeholders."

In this case, the amount minted is set by the
`list_reward` market parameter set in the
`Parameterizer` contract. We'll say more about this
parameter in a future chapter.

Minting also happens when a patron supports the market
by making a payment into its reserve in `EtherToken`
via `Reserve.support()`. We haven't said too much about
patrons yet, but you can think of them as interested
parties who want to help a data market grow and gather
more data (you'll learn a lot more in a [future
chapter](../reserve/index.html)). Patrons may be driven
by altruistic or economic considerations.  The
`Reserve.support()` method provides the mechanism by
which a patron can provide funds to support a market in
return for `MarketToken`. The "algorithmic price curve"
is the protocol mechanism which sets this exchange rate
between the patron's funds and the minted `MarketToken`
(see `Reserve.getSupportPrice()`.)

The third case in which minting happens is when a
datatrust reports that a listing has been queried (We haven't said too much about datatrusts yet. These are the parties which hold the actual data off-chain) via
`Datatrust.listingAccessed()`. The minted tokens are
awarded to the entity which contributed the listing in
question. The actual mechanics of how many tokens are
awarded is a little complicated in this case, so we'll
punt on a thorough explanation till later.


## Burning

What does it mean to burn tokens? Simply put, it's
analogous to a share buyback. Suppose our company has
100 share, 50 of which are owned by other people. If
the company management buys out these other owners and
tears up their share certificates, now there are only
50 shares left. This is a burning operation. 

Burning for `MarketToken` works similarly. There's only
one mechanism for burning, the `Reserve.withdraw()`
method. This allows any stakeholder (that is, any owner
of `MarketToken`) to destroy their `MarketToken`. Why
on earth would anyone do this? Well, it's like a
buyback. Burning your tokens this way entitles you to a
fraction of the funds under the data market's control
(the `Reserve` we've been seeing is basically a "bank
account" of sorts that holds the data market's working
capital). When you burn your tokens, you can withdraw
the proportion of funds from the "bank account" that
you are entitled to. This operation is interesting
because there are no limits. Anyone call call this
function at any time, which allows for interested
parties to cash out at their wish. This lowers the risk
for a potential patron or other data market
participant, since they can always exit if they don't
like the direction a data market is heading towards.

## Units

The `MarketToken` is denominated in "market wei". A wei
is a unit borrowed from Ethereum. A wei is a
billion-biollionth of an ETH (`1/10**18`). As with ETH,
a "market wei" denominates `1/10**18` of a
`MarketToken`. Using wei units throughout prevents
rounding error propagation and keeps contracts simple.

## Comparing with Token Curated Registries

Those of you who have some experience in crypto might
wonder if `MarketToken` is something like a [token
curated registry](https://medium.com/@tokencuratedregistry/a-simple-overview-of-token-curated-registries-84e2b7b19a06). There are some similarities here, since a data market also manages an on-chain "registry."

While our original
[research](https://arxiv.org/abs/1806.00139) on data
market design used token curated registries as a design
guide, the current design is quite different. In
particular, note the contrast of `MarketToken` with
token curated registries tokens, which don't have a
mechanism for minting and burning their associated
token.

## Last Thoughts

Now that you understand `EtherToken` and `MarketToken`
better, it's time to start thinking about how these
pieces fit together into the market. The first place to
start is with `Voting`, which governs all important
changes in the market. You'll learn more in the next
chapter.

<div style="text-align: right"> <a href="../../docs/voting">Next Chapter</a> </div>
