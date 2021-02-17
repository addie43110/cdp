# Exercise Sheet 9

## Buffer with two elements
- implement a buffer with capacity 2 using STM in Haskell
- some code from the legend himself, Miko

```hs
import Control.Concurrent
import Control.Concurrent.STM

type Buffer a = TVar ([a], Int)

-- Creates a new buffer with a capacity of two
newTwoBuffer :: STM (Buffer a)
newTwoBuffer = newTVar ([], 2)

writeBuffer :: (Buffer a) -> a -> STM ()
writeBuffer buff val = do
    (elems, count) <- readTVar buff
    if count > 0 then do
        writeTVar buff (val : elems, count - 1)
    else
        retry

readBuffer :: (Buffer a) -> STM a
readBuffer buff = do
    (elems, count) <- readTVar buff
    case elems of
        []       -> retry
        (x : xs) -> do
                        writeTVar buff (xs, count + 1)
                        return x

-- Tests that a two-buffer can only accept two subsequent writes
testWrite :: (Buffer Int) -> IO ()
testWrite buff = do
    atomically $ writeBuffer buff 1
    putStrLn "wrote 1"
    atomically $ writeBuffer buff 2
    putStrLn "wrote 2"
    atomically $ writeBuffer buff 3
    putStrLn "wrote 3"

-- Asserts that a read on an empty buffer correctly suspends
testReadEmptyBuffer :: (Buffer Int) -> IO ()
testReadEmptyBuffer buff = do
    val <- atomically $ readBuffer buff
    putStrLn ("Read " ++ (show val))

testBoth = do
    buff <- atomically $ newTwoBuffer
    forkIO (testWrite buff)
    getLine
    testReadEmptyBuffer buff
```

- Miko's code is nice because it easily extends to buffers of any capacity
- however, check out the sample solution:

```hs
import Control.Concurrent.STM

type Buffer2 a = TVar [a]

newBuffer2 :: STM (Buffer2 a)
newBuffer2 = newTVar []

readBuffer2 :: Buffer2 a -> STM a
readBuffer2 buffer = do
  xs <- readTVar buffer
  case xs of
    [] -> retry
    (y:ys) -> do
      writeTVar buffer ys
      return y

writeBuffer2 :: Buffer2 a -> a -> STM ()
writeBuffer2 buffer elem = do
  xs <- readTVar buffer
  case xs of
    (_:_:_) -> retry
    _       -> writeTVar buffer (xs ++ [elem])
```

## Semaphore using STM
- implement a semphore using STM in Haskell
- implement the following functions:
  - `newSem num`
  - `p` to acquire the semaphore
  - `v` to relese the semaphore
  - `l` to lookup the number of processes still permitted to ccess the semphore- we `retry` if we try to read the semaphore but it has already reached its limit

```hs
module Semaphore where

import Control.Concurrent.STM

type Semaphore = TVar Int

newSem :: Int -> IO Semaphore
newSem n = newTVarIO n

p :: Semaphore -> STM ()
p sem = do
  n <- readTVar sem
  if n <=0 then do
    retry
  else do
    writeTVar sem (n-1)

v :: Semaphore -> STM ()
v sem = do
  n <- readTVar sem
  writeTVar sem (n+1)

l :: Semaphore -> STM Int
l sem = readTVar sem
```

## Chan implementtion in Haskell STM
- in our first implementation of Chan in Python, we had a bug with respect to `isEmpty` and `unGet`
- does Haskell `Chan` also have this bug?
  - Haskell `Chan` uses MVar, so it would have this bug. But `isEmpty` and `unGet` are not provided by Haskell
- transfer the original Python implementation to Haskell STM (reuse the STM MVars from the lecture)
  - does it still contain bugs for `isEmpty` and `unGet`? the answer is no! bugs are gone!
  - this lot of code is largely the same as the python implementation
  - however, we define a couple of data types:
  - `data ChanEntry a = ChanEntry (MVar (a, ChanEntry ))`
  - `data Chan a = Chan (MVar (ChanEntry a)) (MVar (ChanEntry a))`
  - note here that a `Chan` is two MVars, one is the readHead and the other is the writeHead
  - each Chan entry is a tuple which contains a value and the link to the adjacent chan entry

## Testing STM
- this ws kinda crazy...
- implement an Erlang application which can be used to test different STM implementations
  - number of threads, number of TVars and number of read and write accesses (among other useful parameters you may think of) should be set
  - check correctness of an execution (eg. by checking the sum of all TVar values at the end)
  - extend the STM implementation from the lecture with a monitoring feature (should protocol (?) the number and order of the transactions as well as all the rollbacks and retries that take place
  - try to force an interesting behaviour with your application and watch the result (???)
- i will not go through this exercise as it was extensive and unlike most of what we covered in the lectures
