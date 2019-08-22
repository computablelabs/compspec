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

## Data Flooding Attack

Attackers attempt to flood market with low-quality
listings This attack is mitigated by the enforced
council vote needed for listings to be listed.


## Council DDoS 

Attackers overwhelm the council with a glut of candidates

## Datatrust Fraud 

This attack happens when a datatrust commits fraud and doesn't serve
its purpose correctly.
