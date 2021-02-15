# Exercise Sheet 8

## anbncn Turing Machine
- implement a Turing machine for anbncn
- in this implementation, we used the two stacks (`blank_state(P)`) as implemented in lecture
- my solution had 13 possible states
  - `delta(State, Char)` went through all possible chars and returned
    - `reject`
    - `accept`
    - or `{NewState, NewChar, Direction}` where `Direction` was either `l` or `r`
- there are two stacks which represent the left and right "halves" of the Turing tape respectively
  - in this implementation, we ALWAYS read from the right stack

## Turing Machine
- implement a Turing machine as a bunch of processes which hold their current letter and the pids of their neighbours
- pretty straight forward

## Stacks by using the mailbox
- implement a stack by using the mailbox of a process
- this explains it succinctly
  - "We use the mailbox to implement the stack. To realise `push` we just send messages to the stack process. But because the mailbox is a queue and not a stack, we need to go through the whole message box to realizse a `pop` returning the last message. To do this we send a message `top` to our self, to be able to recognize the end of the mailbox. We go through the whole mailbox sending ourselves ll the messages again if the `top` message isn't the next one. If the `top` messages followsa message, we return this message."
  - note that we therefore need to keep track of the last messge popped in this implementation
  - initially the is set to a `blank`, so that if we immediately read `top`, we just return a `blank`

```erl
stack(P) ->
  receive
    pop -> self() ! {top, top},
           stack_pop(P, blank)
  end.

stack_pop(P, Last) ->
  receive
    {Type,Value} -> case Type of
                      top   -> P!Last,
                               stack(P);
                      value -> case Last of
                                blank -> ok;
                                _     -> self()!{value,Last}
                               end,
                               stack_pop(P,Value)
                     end
  end.
```

## Accounts using Concurrent Haskell and STM
- define a function `collectedLimitedTransfer :: [Account] -> Account -> Int -> IO Bool`
  - transfers money from a list of accounts to a single account
  - if there is not enough money in the lists of accounts, the transfer does not happen
  - return value shows if the transfer was successful
- because i am so bad at Haskell, I am going to retype the sample solution

Naive Solution:

```hs
collectedLimitedTansfer :: [Account] -> Account -> Int -> IO Bool
collectedLimitedTransfer froms to am = do
  success <- collectedWithdraw froms am
  if success then deposit to am else return ()
  return success
  where 
    collectedWithdraw :: [Account] -> Int -> IO Bool
    collectedWithdraw [] _ = return False
    collectedWithdraw (from : froms) amount = do
      bal <- takeMVar from
      if bal >= amount
        then do
          putMVar from (bal - amount)
          return True
        else do
          success <- collectedWithdraw froms (amount - bal)
          putMVar from (if success then 0 else bal)
          return success
```

Note that this implementation runs into a deadlock if an account is in the list of source accounts twice
