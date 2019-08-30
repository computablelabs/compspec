# Attacks

This chapter introduces you to known attacks on the
Computable protocol and known defenses against such
attacks.  When developing Computable, we made some
trade-offs. In particular, we've consistently voted for
simplicity. This makes it possible for us to actually
ship working code in reasonable timeframes. But our
choice to go with simplicity means that there are some
more sophisticated attacks that bad actors could
execute against your data markets. These aren't
necessarily easy to do, but forewarned is forearmed and
we want to give you the best knowledge we have. Let's
get started.

## Front Running Attacks

In the current version of the Comptuable protocol, it
is possible for a determined adversary to front-run a
number of protocol interactions. What does that mean?
Recall that Ethereum transactions are broadcasted to
the network of Ethereum nodes at large. That means it's
possible to detect a transaction while it's being
propagated and before it's confirmed on the chain as
permanent. A front running attack is when an attacker
detects such an unconfirmed transaction and sends an
alternative transaction which exploits the information
it's gained before this transaction goes through. This
is possible if the attacker runs a miner for example,
or if it's willing to pay high gas cost to incentivize
miners to pick up its transaction before yours.

How could this work on Computable contracts? Here's an
example: if a listing with a given `listingHash` is
proposed but not yet confirmed as a transaction, the
adversary could create an alternative transaction with
the same `listingHash` to attempt to block the original
listing.

The same attack can be applied to any candidate,
allowing a determined attacker to bring a market to a
standstill by preventing all governance actions from
going through. However, this attack requires some
infrastructure, with the attacker having to
consistently monitor in-coming transactions and
front-running them, possibly by expending more on gas.
This means that the attack is pretty hard to execute,
but definitely within the realm of possibility.

## Datatrust Fraud 

This attack happens when a datatrust operator commits
fraud and doesn't serve its purpose correctly. At
present, this is a known failure mode of the current
version of the protocol.  However, this topic is being
actively researched on the
[forums](https://forum.computable.io/). Future versions
of the Computable protocol should lower the trust
requirements for the datatrust. In addition, future
versions of the protocol will allow multiple datatrusts
to serve a data market, making markets more robust
against fraudulent datatrust operators.

## Reserve Race Conditions

Calls to `Reserve.support()` and `Reserve.withdraw()`
will be publicly visible on the Ethereum network before
they are confirmed. A dedicated watcher could watch for
incoming large `Reserve.support()` orders, front-run by
buying some amount itself with `Reserve.support()`,
then immediately calling `Reserve.withdraw()` after the
large order is confirmed.

The `spread` provides some defense against this type of
attack, since the watcher would have to pay for the
spread when executing this attack. As a result, only
large market shifts would be vulnerable to these sorts
of attacks.

## Candidate Flooding Attack

At present, there is no stake required to propose a new
candidate (remember that candidates can propose new
listings, reparameterizations, or change of datatrust).
This means that it will be relatively easy for a
malicious party to spawn many candidates. This party
could then wait till the last minute to vote for one of
its candidates. If market governance isn't actively
tracking for such last-minute votes, it's possible that
a such a candidate could slip through and be accepted
(particularly since there isn't a quorum requirement
for votes at present).

If the vote is for reparameterization or change of
datatrust, the attack could prove to be disruptive. At
present, there isn't a mitigation for this attack
beyond careful monitoring by market stakeholders. You
might ask why this situation arises. The market smart
contracts were designed to be maximally simple. This
meant that we made trade-offs that preserved
simplicity. However, this simplicity opens the door to
some sophisticated attacks. If these attacks prove to
be serious in practice, future versions of the protcol
will likely alter voting to make critical changes
significantly harder.

There are a number of specialized types of candidate
flooding which we will explore in more detail below:

### Data Flooding Attack

Attackers attempt to flood market with low-quality
listings. This is a special case of a candidate
flooding attack where the candidate is a listing
candidate. This attack is possible to partially defend
against by an honest datatrust which can set up
defenses to prevent obvious DoSing attempts.

### Reparameterization Attacks 

Attackers can flood the market with many
reparameterizations. If a nonsensical
reparameterization makes its way through, it's possible
that the market could be bricked. In particular, an
attack which sets the voting stake exorbitantly high
could freeze market governance and make it impossible
to further change market stake.

## Last Thoughts

We've listed a number of known attacks in this chapter,
but the one thing to remember about security is that
there are always unknown unknowns. We've done our best
to secure the Computable smart contracts and systems,
but there might be unforeseen subtle bugs. When you
work with crypto, there is always a risk. Perform your
own analysis. If you detect anything wrong, share with
the broader Computable community.

In the next chapter, we'll provide you some tips and
tools for continuing your journey in the Computable
ecosystem.

<div style="text-align: right"> <a href="../../docs/continuing">Next Chapter</a> </div>
