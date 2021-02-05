# Lecture 3

## `RLock.py`

### `acquire()`
- if `self.holding_thread == threading.currentThread()`, simply increase the counter
  - note there can only be one thread which will pass this check
- if `holding_thread != currentThread()`, we want to acquire the lock
  - then we can increase the counter and update the `holding_thread`

### `release()`
- if a thread tries to release the lock and it is not the `holding_thread`, we should raise an error: `cannot release un-acquired lock`
- otherwise, if the counter is 1:
  - set `holding_thread` back to 1
  - set counter to 0
  - release the lock
- if the counter is not 1, simply decrease the counter

Points:
- there is no critical section between the `acquire()` else-case and the `release()` else case, because both can only be accessed by the same thread, and the same thread can only do one part at a time. 
- release the lock LAST so no one can get in before
- the overhead of using an RLock instead of a regular lock is not that large.

**Note:** `tryAcquire()` in Python can be achieved with `acquire(False)` (`blocking=False`). This works for both Lock and RLock.

## Producer-Consumer Problem
- have several producers, which all produce the same product
- we have several consumers, which consume the product
  - if there is no product left, the consumers should wait (suspend) until product is available again
  - in code, you say, 'I want to wait until this changes or this happens'
- we need to reuse the blocking mechanism to wait for the product
  - the consumer needs to acquire, and the producer needs to inform by means of release

### `prodcon.py`

|**Function**|**Description**|
|---|---|
|`producer()`|simulates a produce which loops infinitely, sleeping and adding to the store|
|`consumer()`|also loops infinitely, sleeping and consuming from the store. need to write `store[0:1] = []` to mutate the store-list|

Points:
- the consumer should check whether there is anything to consume
  - if we do this with a simple `while len(store)==0`, we both perform busy waiting and the check and taking from the store are not atomic, meaning we can run into critical errors
  - instead, this should be done with a condition variable (`cond=threading.Condition()`
- condition variables wait until someone else releases the cond, then they must acquire the variable (only one will acquire it!)
  - `notify()` will wake all sleeping threads up, but they cannot do anything until the lock is released and available for taking.
- **EXAM QUESTION:** why do we need to use a `while` loop when using Condition Variables???

#### Condition variables in Python
```Python
import threading

cond = threading.Condition()

with cond: # always need to take the condition variable
           # before you can use it
  cond.notify()

with cond:
  while len(store)==0:
    cond.wait()
```

## `Condition.py` (first round)

|**Attribute**|**Type**|
|---|---|
|sync|our own lock for synchronization|
|waiting_q|lock to simulate the waiting block and notify|

```Python
def wait(self):
  self.sync.release() # must release cond lock while waiting
  self.waiting_q.acquire()
  self.sync.acquire()

def notify(self):
  self.waiting_q.release() # problematic, if no thread is waiting
                           # it destroys the initialization
```

## Race conditions
- when you have a race condition, the result of the program depends on the execution order of the threads
- surprisingly, they aren't always bad! for example, two consumers want to by the last TV on amazon. only one can get it, depending on who is faster.
