# Lecture 4

## `Condition.py`
Points:
- we will check if `self.sync` is locked first
  - if not, error: `cannot notify/wait on unacquired lock`
  - however, since we have release the sync lock before acquiring the `waiting_q` lock, there is no synchronization there anymore (in `wait()`)
- another solution would be to use semaphores

### First solution: `waiting_q` as a list (round 2)

- the first solution is to use separate `waiting_q` locks for each waiting/sleeping thread
  - `self.waiting_q = []` (becomes a list)
- whenever someone waits, we create a new lock, acquire it (to make it unavailable), and add it to the list
  - that way, we can stay locked under the `sync` lock while we do all the critical things
- in notify, we need to check if anyone is waiting in the list
  - if there is, we release the corresponding lock and remove it from the list
  - if no one is there, we simply do nothing
- do the problems we had in round 1 still exist?
  - no. even though we still have unsynchronization after `self.sync.release` in `wait()`, if we notify before the thread is fully 'sleeping', it will simply continue immediately, without needing to sleep

## `Account.py`
Idea: simulate a bank account. Allows for depositing, withdrawing, and transfering. Show that with transfering, we can run into problems if we are naive.

Looking at the transfer method more closely...

```Python
def transfer(self, other_acc, amount):
  if self.withdraw(amount):
    other_acc.deposit(amount)
    return True
  else:
    return False
```
- This works, but the withdraw and deposit are completely separated (not atomic)
  - if we observe the state in between the withdraw and deposit, we can observe that money is no longer there, but does not exist anywhere else yet.
- so we want to make the withdraw and deposit one atomic action
  - we can do this by taking our `self.lock` and the `other_account.lock`, but now we can run into deadlock
  - two people try to transfer to each other at the exact same time
  - can solve this two ways: 
    - use `acquire(False)` for non-blocking lock
    - enforce some global ordering for accounts, such as account number

## Message passing in Python

### `Mvar.py`
Idea: buffer of capacity one. Can put and take

|**Function**|**Description**|
|---|---|
|`put(v)`|if mvar is filled, then suspend, if mvar is empty, then fill with v|
|`take()`|if mvar is filled, then content is removed and returned, if mvar is empty, then suspend|

Points:
- Mvar should contain content and a condition variable
- in `put(v)`, while mvar is full, wait on the condition variable
  - then set the content value and notify if any other threads waiting for a filled mvar
- in `take()`, pretty much the same thing, also notify at the end for any threads waiting for an empty mvar
- however, it is possible to notify the wrong thread by doing this
  - the solution is actually the use `notifyAll()` instead of a simple `notify()`

