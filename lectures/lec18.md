# Lecture 18

## `MVarSTM.erl`
Idea: implement MVar in Haskell so that it works with STM

- we will define our MVar as a TVar which `maybe` holds a value
- during `tryPutMVar`, we introduce the notion of `orElse`
  - `orElse` will execute a transaction and if it runs into a `retry`, it will execute a different transaction

## Implementing STM
- in databases, you usually use pessimistc locking
  - this is not so appropriate for software transactions
  - we want to execute processes as much in parallel as possible and do not want to block all other processes when executing one
- instead, we use an optimistic strategy
  - we run as much in parallel as possible and restart when we encounter a problem
- in order to do this, we should not immediately write into TVars when we encounter a `writeTVar` in a transaction
  - instead, we should keep track of all TVars with write requests until the end of the transaction
  - then we write to all of those TVars in one commit phase
- it may turn out that a transaction is invalid when we go to validate it
  - before committing, we need to check whether all read TVars are still valid
  - thus, we may not commit it
  - instead, we will have to rollback the whole transaction
- we must collect both a ReadSet and a WriteSet during execution
  - readset is used for a validity check
  - writeset is used for the commit
- we must also consider how to avoid deadlocks in the locking phase (commit phase)
- we will put back all locks taken if any lock is not available
  - after putting back all locks, we will have to try again

## stm.erl
Implementing `STM` in Erlang.

In each process in Erlang, we get local storage (a dictionary), accessible by `get(K)` and `put(K,V)`.

`atomically(Transaction)`
- first, we initialize an empty RS and WS
- then we run the transaction
- this can result in three possibilities: we catch a rollback, we catch a retry, or everything was fine and we get the result (return value) of the transaction
- case rollback: we should recurse (aka restart) because someone else committed while we were executing our transaction
- case Res: we need to validate the RS, and if it is invalid, gotta restart the transaction
  - if everything is still valid, we quickly commit and then return Res
- see lecture 19 for what happens when we run into a retry

`new_tvar(V)`
- in our implementation, TVars will simply be processes
- TVars handle reads and writes
- TVars also need to be locked, which means we should have two versions of the TVar
  - `tvar_unlocked` and `tvar_locked`

`read_tvar(TVar)`
- read the tvar,
- if the tvar is already in our readset, check if our stored value matches the one just read
  - if it does not, we should check to see if we ourselves wrote to that tvar by checking the write set
  - if we already wrote to that tvar in our write set, we can simply return the value stored in our write set
  - note that in that case, we do not have to add the variable to our read set, since it will just be overwritten by our writeset when we commit anyways


`write_tvar(TVar, V)`
- we simply add the tvar and the value to our writeset
- remember, we don't actually write the TVar here!

