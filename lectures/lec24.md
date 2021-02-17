# Lecture 24

- in STM you can also establish invariants, which check properties of your transactions
  - eg. check if two variables always contain the same value or if the amount of an account is positive
- there is a function called `always :: STM Bool -> IO ()`
  - the argument is a transaction which checks properties of system state
  - it returns True is the state is okay, false otherwise

### checking if the amount of an account is positive (example)

```hs
main = do
  acc <- new_account 0
  atomically $ always $ do
    bal <- readTVar acc
    return bal >= 0
```

Assume invariants are always introduced with the direct combination of `atomically` and `always`.

When should we check an invariant?
- before another transaction commits (in the validation)
- when establishing a new invariant; if the invariant does not hold, throw an exception

We will store every invariant within a global state, such that they can be checked before every (?) commit
  - an invariant must only be checked if the WS of a transaction contains TVars read in the last execution of that invariant
  - the check has to consider the WS of the transaction but without writing them to the TVars
  - write_tvar actions in the invariant should not be committed (although it would be strange to have writes in the invariant in the first place)
- if the invariant yields true:
  - commit the transaction and update the RS of the invariant for the next invariant initiation
- if false:
  - rollback the transaction (or break it by an exception)


## Open problems in STM
- garbage collection
  -  in our implementation, no TVar becomes garbage
- often lists should be replaced by balanced search trees
- we lock all TVars with respect to a global order
  - some experiments show that putting back the locks if you don't get the next one can be much more efficient in independent concurrency
- Haskell's type system avoids side effects within transactions; this is not guaranteed in our Erlang implementation

## Transactions als occur in distributed settings
- distributed data base (we will discuss distributed commit protocolls next lecture)
- web applications assume transactions as well
- consider a webserver (such as amazon)
  - we have state going from one step to another
  - ie. you put something in your shopping cart, it should stay there
- but we can see that every webpage uses transactions
  - each browswer obtains a webpage
  - data is sent from the webserver to the browser
  - then we update the data and send it back
  - before the webserver commits this data, we need to check if it is still consistent with the current state
-scenario:
  - application with user management; only admins can delete users
  - problem: if a user edited their profile and submitted it, a deleted user was reactivated there was no validation check in place
- solution: the framework of the webserver supports transactions which run over multiple client-pages
- usually, exceptions are used instead of rollbacks
  - we can show these exceptions to the user and inform them that their state is no longer valid

More shopping basket talk:

- The shopping basket is usually a state of a transaction
  - but the things inside the basket are not yet yours!
- there are read/writes on resources like TVars within the transaction 
  - eg. decrementing the number of items when the number is very large
- however, our approach would show that one transaction has to rollback instead of simply sequentializing the two decrement transactions
  - it doesn't matter if the data was a bit invalidated in the middle!
- this idea of improvement can also be used in STM
