# Lecture 6

## Learning erlang recap
Nothing too important...

Points:
- mailboxes can only be read by its own process
- messages are stored in an almost FIFO-Queue

### `store.erl`
Idea: A simple store program to demonstrate message passing in Erlang.

Points:
- demonstrate `receive`, which will block if there are no messages to be read
  - pattern matchin in receive very useful
- demonstrate `spawn` for spawning new processes
- demonstrate having both a 'server'-like process, and aclient-like process which can parse input from the user
- show `after 200 -> smthin()`

