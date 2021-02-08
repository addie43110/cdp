# Lecture 12

## Linda (coordination language)
- an idea on how distributed programming can be realized
- in Linda, you have a server called the Linda-Server or Tuple Space
  - inside, can store any arbitrary tuple
  - the same tuple can be stored multiple times
- there are two main functions:
  - `out(Tuple)` sends a tuple into the tuple space
  - `in(Pattern)` returns a tuple matching the pattern, and deletes that tuple from the tuple space
    - if no tuples match the pattern, this call may block
  - `rd(Pattern)` returns a tuple matching the pattern, but does not delete the tuple from tuple space
- there are variants with timeouts, so tuples in the tuple space only stay there for a certain amount of time
  - rd calls and in calls timeout after a while, don't block, etc.

## Implementing the tuple space in Erlang
- linda server will be a process with a list of tuples
- how can we realize pattern matching?
  - Patterns are not first-class citizens in erlang, so cannot bind them to variables or send them as arguments
  - instead, use a function!
  - ie. `fun({hi,X}) -> {hi,X} end`

## `linda.erl`

`tuple_space(Data)`
- can receive out message
- can receive in message
  - `{in,PatternFun,CPid}`
  - run `pattern_check()` and return the value (if there is a match)
  - if nothing, don't send any message back! client will suspend
  - then we can add the clientPid and the pattern to a list of suspended `in` clients (Huch calls them in requests,`Reqs`)
    - important to distinguish between in and rd requests, since rd are non-destructive!
- can receive rd message
  - similar to in but should not delete the tuple from the space

`pattern_check(PatternFun, Tuples)`
- take the pattern function an apply it to each tuple in the Tuple list
- if there is an error (`{'EXIT',_}`), then keep checking
- if some result, return `{just, Res}`
  - should also delete this tuple from the list of tuples
  - then if we keep track of our modifies tuple list, we can send it back as the new tuple list for tuple space to use
- if we run out of tuples to look at, return `nothing`
