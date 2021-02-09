# Lecture 17

## Concurrency in Haskell

### `Account.hs`
Idea: reimplement our account code in Haskell to show how concurrency works in Haskell

To make a new data type binding, we can use `type` which basically creates an alias

```hs
type Account = MVar Int
```

When using an MVar (or concurrency, I think), we always need to use the `IO` monad.

```hs
newAccount :: Int -> IO Account
newAccount bal = newMVar bal
```

Haskell `do` allows us to chain together multiple instructions in one function.

`forkIO` is Haskell's `spawn()`

```hs
forkIO (someFunction arg1 arg2)
```

## ACID

- transactions in databases are sequences of read and write operations which should fulfill the ACID properties
- A: atomicity
  - each transaction is executed atomically
  - maybe it's not *really* atomic, but you can't observe that is isn't
- C: consistency
  - can't observe in-between states
- I: isolation
  - for the programmer, when you execute a transaction, you can pretend that you are the only programmer in the world
  - other transactions have no effect on your code
- D: durability
  - if a database crashes, we can still recover
  - changes in the databse stay consistent over time, independent of the system crashing
  - this is not relevant to this course

### Software Transactional Memory (STM)
- uses ACID for communication between concurrent processes / threads
- only durability is not guaranteed, since we are assuming concurrency (ie. all threads operating on one computer)
  - thus, if the computer crashes, everything is lost
- STM uses TVars instead of MVars
  - TVars are like MVars but they can never be empty
  - we will use `read` and `write` to Tvars within the `STM` monad
- STM actions can be executed within the `IO` monad by using the function `atomically :: STM a -> IO a`
- must `import Control.Concurrent.STM`

### `AccountSTM.hs`
Idea: reimplement `Account.hs` using the `STM` monad and TVars

- thus, all of our account functions are changed from `IO a -> STM a`
- when we execute multiple `STM` actions together, we can use `atomically` to execute it as one action (or seemingly one transaction)

## Back to STM...
- everything becomes much more composible
  - we can reuse functions without problems
- we can combine everything and we don't need to worry about locks
- our entire transaction will be executed atomically and we don't need to worry about other transactions out there in the world

## Also a part of this lecture:
- `phils.hs`
- `philsSTM.hs`


Note:
- when we try to take the state using the `STM` monad and TVars, we first check if it is available by reading it
- if it is available, we take the stick (`writeTVar stick False`)
- however, if it is unavailable, we need to essentially wait until the stick is available and we can try to take it again
- we could just keep calling `takeStick` but this performs busy waiting
- instead, we will use `STM` action `retry`, which essentially waits until one of our read TVars has been written to, and then will re-execute the transaction
  - this ensures that we do not perform busy waiting
