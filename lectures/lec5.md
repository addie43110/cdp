# Lecture 5

## `Mvar.py` (round 2)
- there was a problem with Mvar.py because the notify's could notify the wrong thread
  - consider two producers and two consumers starting with an empty mvar
  - the solution is to use notifyAll
  - however, really there is no reason for a put thread to notify itself. it only ever needs to notify a suspended take thread
  - thus, we can split our one condition variable into two, each for their own purpose
  - at first, when we do this, it looks like we can have deadlock,
    - due to our check on self.empty, the boolean value can only be one value at a time, so deadlock is not possible

## Channel

- channel is an unbounded buffer
  - can `write(v)`: add value to the end of the channel
  - can `read()`: return oldest value. if channel is empty, suspend
- because channel is FIFO, we should use something like a queue (eg. linked list)
- if we use a linked list, each list element can be an MVar, and the read and write heads can be MVars themselves
- all reads and all writes must be sequentialized (each read action can only read one element. even if 10 read operations, only one will get a certain element.)

### `Chan.py`

- by using Mvars for our read and write heads, we can easily sequentialize these actions
- by using Mvars as elements, reads will suspend when channel is empty
- Frank says, 'elegant'
- I say, slightly confusing, but seems nice
