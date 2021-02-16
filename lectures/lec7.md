# Lecture 7

## `store.erl` recap
`delete()` is added to the store

## `phil.erl`
- now, each stick is a server
  - runs infinitely and has state
  - in Python, sticks were objects (Locks), but in a functional language, we can run it as a process / server
- `new_stick()` will spawn a stick (server) which is avaiable
  - if the stick is taken, we switch to a stick unavailable server
- to block, we can use `receive`

Code for executing someting _n_ times:
```erl
Sticks = lists:map(fun(_) -> new_stick() end, lists:seq(1,N)).
```

- still run into deadlock when programming the dining philosophers progam in erlang
  - every philosopher sends the take message, then suspends on receive because they are all unvailable
- to fix this, we can easily change the order of the 5th philosopher's sticks
  - alternatively, we can check if sticks are avaiable or not before we attempt to take them

## MVars in Erlang (`mvar.erl`)
A simple erlang program implementing MVar.

- put doesn't block unless we wait for a message from the mvar server

## `phil_mvar.erl`
- we can use MVars to implement the Dining Philosopher's problem
- each stick becomes an MVar
