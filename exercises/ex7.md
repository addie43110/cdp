# Exercise Sheet 7

## Using LTL
- write an LTL formula that shows how the first implementation of `Chan` using `MVars` was incorrect for `isEmpty`
  - remember, the first implementation for `isEmpty` was to first read the ReadHead then the WriteHead and compare if they were the same
  - if we had a Read action queued up first trying to read an empty MVar, then our isEmpty would forever be blocked
  - thus, the LTL formula should state that when invoking the function `isEmpty()`, we should always eventually terminate; that is `isEmpty()` cannot block forever
  - `G(isEmpty -> (F(not isEmpty)))`
  - we can set the property `isEmpty` when we first enter the function and unset it as the very last thingwe do before we exit the function
- write an LTL formula to test the broken behaviour of the serverless chat
  - here, all clients should have the exact same client list at all times
  - furthermore, if a client C1 knows C2, and C2 knows some client C3, then C1 should also know C3
  - `G({knows,C1,C2} and {knows,C2,C3} -> {knows,C1,C3})`
  - in the solution, Huch also used the fact that clients should be loggedIn before they can know of each other

## Missing LTL operations and global propositions
- extend the LTL implementation by Until, Release, and Weak Until
  - the following functions must be extended: `showLTL`, `normalize`, and `check`
- add a function Fn(Phi) which checks if Phi is valid after n steps
  - turns out Fn(Phi) is implemented just as another LTL function
  - `fn(N, Phi) -> (fn, N, Phi).`
  - thus, add it to `normalize` and `check`

```erl
check({fn,0,Phi},Props) -> check(Phi,Props);
check({fn,N,Phi},Props) -> check(disj(Phi,x(fn(N-1,Phi))),Props);
```

## Verbose LTL tests
- create a counter to show how often Finally and Globally are separated
  - here, we just add a counter to the finally and globally tuples:
  - `f(Phi) -> f(0,Phi).`
  - `f(Cnt,Phi) -> {f,Cnt,Phi}.`
  - we extend `showLTL` and `normalize` and in `check`, we increment the counter
  - `check({f,Cnt,Phi},Props) -> check(disj(Phi,x(f(Cnt+1,Phi))),Propts);`
- when an invalid state occurs in a test, print the path to that state
  - here, we add another tuple to the `ltl` process, `fun() -> ltl([],[],[],[]) end`
  - the Path will be composed of which tuples are present in each state
  - thus, at each state, the props in that state are saved as an element in the Path
  - we pass the Path to the `assertion_violation` code so that it can print the path along with the violation
  - we also therefore have to pass the Path to `analyze` since this is the code which directly calls the `assertion_violation` code

## Chat using Linda
- implement a chat in Erlang using the tuple space as chat server
  - must initialize the chat server, which is the Tuple space
  - register under `linda` and set the client list to `[]`
- to start a client node, we must reach out to the `linda` node (`{linda, Node}`) and get the tuple server pid
  - also get the client list
- functions we will need
  - `set_client_list(SPid, [])`
  - `get_client_list(SPid)`
  - `write_message(SPid, {join,Name})`
  - `get_message_by_id(SPid, MessageId)`
    - because we are using the Linda server, we have no mailbox (FIFO) properties
    - thus, we can add a message id to every message and they increase by 1 each time a message is sent
    - this way we can read them in increasing order to simulate a queue
  - `get_and_inc_message_id(SPid)`
