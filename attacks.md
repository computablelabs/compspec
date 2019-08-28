# Attacks
This section catalogs known attacks on the protocol and
known defenses against such attacks.

## Front Running Attacks

In the current version of the protocol, it is possible
for a determined adversary to front-run a number of
protocol interactions. For example, if a listing with a
given `listingHash` is proposed but not yet confirmed
as a transaction, the adversary could create an
alternative transaction with the same `listingHash` to
attempt to block the original listing.

The same attack can be applied to any candidate,
allowing a determined attacker to bring a market to a
standstill. However, this attack requires some
infrastructure, with the attacker having to
consistently monitor in-coming transactions and
front-running them, possibly by expending more on gas.

## Datatrust Fraud 

This attack happens when a datatrust commits fraud and
doesn't serve its purpose correctly. At present, this
is a known failure mode of the current version of the
protocol.  However, this topic is being actively
researched on the
[forums](https://forum.computable.io/). Future versions
of the Computable protocol should lower the trust
requirements for the datatrust.

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
candidate (remember that candidates can be for new
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

[Next Chapter](../userjourney/index.html)
