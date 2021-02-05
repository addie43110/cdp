# Lecture 2

## `time.py` recap

Points:

- we had critical sections and used locks to prevent multiple threads from accessing critical sections at the same time
- RLocks allow for **re-entrant locking**

## `RLock.py`
Idea: re-implementing the RLock class in Python

Points:
- we will need
  - a counter, to count how many time the RLock was acquired
  - a process ID of the holding thread, initialized to `None`
  - a lock in order to create a blocking mechanism
  - a second lock to protect our critical sections
- check if the thread trying to acquire the lock is the same one holding the lock
  - if the counter is 0, initialize the holding thread
- an RLock which has not been acquired cannot be released. this should raise an exception
- if we acquire the sync lock, then try to acquire the lock and block, the sync lock will not be released. we must release the sync lock before acquiring the other lock.
  - but if we release the sync lock before acquiring the other lock, there could be a thread switch and we end up unsynchronized
- *working with multiple locks is always dangerous/difficult*

|**Attribute**|**Type**|
|---|---|
|`counter`|int|
|`holding_thread`|PID|
|`lock`|blocking lock|
|`sync`|critical section lock|
  
|**Function**|**Description**|
|---|---|
|`acquire()`|acquire the RLock|
|`release()`|release the RLock|

## Dining Philosopher's problem, `diningPhil.py`

We will use locks to represent the chopsticks. Taking a chopstick can be represented through acquiring the lock; putting the chopstick back down represented through releasing the lock.

Philosophers will be represented by threads.
- each philosopher should know his left and right stick (and not all the sticks)
- a philosopher never terminates
- a philosopher should think, then eat in an endless cycle

Frank's important thought: 
- _every_ deadlock can be modelled by the Dining Philosopher's problem

### Two solutions to a deadlock
- if a stick is not available, put down all the sticks you are holding.
  - do we need to make sure that looking to see if a stick is available and taking the stick should be done atomically (ie. as one action)?
  - or can we just use something like `tryAcquire()`
  - the answer is the latter, because the last philosopher who tries to take the stick, the second stick will already be gone. "The last philosopher will clear the dangerous situation."
  - however, this is a type of busy waiting. a thread might take one lock, see the other is not available, then release their lock. but if the other lock is not available for 2 hours, the thread might keep taking and releasing again and again.
- enforce a global ordering to the sticks, such that one philosopher picks up his sticks in a different order
