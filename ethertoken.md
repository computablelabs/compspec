# Ether Token

The first point of entry into the Computable ecosystem
is the `EtherToken` contract. This contract is simply a
wrapper for ETH that gives it an ERC-20 interface. You
will need `EtherToken` for all your interactions with
the Computable smart contracts.

To get started, you need to call the
`EtherToken.deposit()` function and deposit some ETH

```
@public
@payable
def deposit():
  """
  @notice Facilitate a user purchasing EtherToken with Eth at a 1:1 ratio
  """
  self.balances[msg.sender] += msg.value
  self.supply += msg.value
  log.Deposited(msg.sender, msg.value)
``` 


This snippet is the first example in
this book of Vyper code. Vyper is a smart contract
language that runs on the Ethereum ecosystem. Vyper's
syntax is pretty similar to Python code so this code
shouldn't be too hard to follow. The basic idea is that
the `EtherToken` contract maintains an internal ledger,
`self.balances`.  Here, `msg.value` is the ETH that
you're sending to the `EtherToken` contract. This
function stores the ETH you give it in `self.supply`
for safe keeping, and awards you an equivalent amount
of `EtherToken` in its ledger.  It ends by emitting an
"event," `log.Deposited` which signals to watchers than
a deposit has happened on this contract.

There was some detail here, but the basic idea is
simple. `EtherToken.deposit()` simply deposits the ETH
you send it and mints you an equivalent amount of
`EtherToken` in a 1-1 fashion. 

Now let's suppose that you've interacted with the
Computable system to your hearts content and made some
healthy capital that you'd like to take out as ETH. In
that case, you simply call the `EtherToken.withdraw()`
function:

```
@public
def withdraw(amount: wei_value):
  """
  @notice Allow msg.sender to withdraw an amount of ETH
  @dev The Vyper builtin `send` is used here
  @param amount An amount of ETH in wei to be withdrawn
  """
  self.balances[msg.sender] -= amount
  self.supply -= amount
  send(msg.sender, amount)
  log.Withdrawn(msg.sender, amount)
```

This function
simply undoes the effect of `EtherToken.deposit()`
which you just saw. It removes the requested amount
from its internal ledger and sends back the ETH by
calling the `send` function. Finally it issues an event
with `log.Withdrawn` to signal that a withdrawal has
occurred. 

If you're new to Vyper, you might be feeling confused
here. What if `amount` is bigger than my balance? Can I
steal money from the system? Luckily, Vyper protects
against these types of attacks. `self.balances` stores
values of type
[wei_value](https://vyper.readthedocs.io/en/v0.1.0-beta.11/types.html#wei).
This is an unsigned type. A subtraction that would
cause a negative value will consequently throw an error
and fail.

The ease of doing math in this fashion is one of the
major advantages Vyper has over other languages such as
Solidity, which require more verbose safe math
libraries to achieve similar functionality.

[Next Chapter](../markettoken/index.html)
