# Lecture 21

## STM strategies

- optimistic
  - record all read and written TVars
  - when transaction is over, validate the RS and commit the WS
  - locking only during the validation and commit phase
  - if invalid, rollback
- pessimistic
  - lock TVars as we read them so we don't have to validate them later
  - when we lock the TVars, no other thread can read or write to them
  - problems with locking like this is it's easy to naively encounter a deadlock
  - to avoid deadlock, we must release all locks if any lock cannot be acquired
  - even when we release the locks, we can still run into a live lock situation where two processes see the other is using a lock they need, so they put their locks down and start again
  - they pick up the first lock, see the other is locked and lay back their lock
  - this happens again and again and again

## `stmPes.erl`
- when we get a read request, we should lock the TVar
- if we get a read request when the TVar is already locked, we should notify the asking process that the TVar is locked
- if there is a request to lock when the TVar is already lock, again we should notify the asking process  (with `is_locked`)
- note that having a version number is unnecessary in the pessimistic approach
  - when we read a TVar, we immediately lock it, so it's not possible tht our read TVars become invalid
  - thus, we never need to validate by checking version numbers
- if we receive a rollback during the execution of our transaction, we need to gather our WS and RS and free all locks (if you locked the write TVars immediately)
- it also makes sense not to rollback until one of the TVars we locked (including the one we just tried to lock) iscfreed (unlocked)
