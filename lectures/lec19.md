# Lecture 19

## `philsSTM.erl`
Idea: test out our STM code by implementing the dining philosopher's problem using our `stm.erl`

- by attempting to implement `take_stick(Stick)`, we realize we have not yet implemented `retry()` in `stm.erl`, so our program will do busy waiting
- but even though we do busy waiting, our STM program works!

## `stm.erl`
We will now implement `retry()` so that we do not run into busy waiting

- we know we branched on certain TVar values before we reached a retry
- thus, we should wait for the TVar values that we read to be written to before we actually retry the transaction again
  - clearly, if the TVars are not changed, we will end up on the same branching path which leads to the previous `retry`
- the solution is to connect our Pid to all the TVars we have read
  - then if any of those TVars is written to, we must unenroll ourselves from all and try the transaction again
- we must be careful that we don't obtain too many `restart` messages from the TVars we have connected too
  - to solve this, we create a separate process which only receives one message before terminating
  - this ensures that `restart` messages don't stack up in our mailbox
- also consider that we are linking ourselves to the TVars and there is a switch
- then another process commits a TVar that we haven't yet linked to
- already we are inconsistent
- thus, we must lock all TVars (in the RS) and validate the RS before linking to the TVars
  - don't forget to unlock!
