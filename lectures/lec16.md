# Lecture 16

## Erlang is Turing Complete

Termination of Erlang code cannot be decided for the following reasons. We argue that the corresponding aspects of Erlang can be used to model a Turing machine.

- Erlang provides data structures, which can encode a Turing Tape
  - a list can be of arbitrary size and thus can represent the tape, at least the used finite part of the tape
  - the same holds for other data structures like tuples
- if we don't use data structures, we could also use numbers
  - having two counters allows for the encoding of a Turing machine
  - could have one be the tape content, and one is the position of the head
  - or could have that one is the left tape content, and the other is the right tape content, thereby giving the position of the head implicitly
  - moving the head therefore means dividing and multiplying by two (and maybe incrementing/decrementing)
- Erlang provides an arbitrary number of processes
  - each process acts as a tape cell which knows its value and its left and right cell
- Erlang provides two (we will use three) processes in combination with recursion
  - each processes holds a stack (the runtime stack) so we can model the left and right side of the tape again
  - this one implemented in lecture
- Erlang provides a mailbox of arbitrary size
  - mailbox can also be used to encode the tape


## `tm_rec.erl`
Idea: implement the three-process Turing machine. Two processes are stacks.

`stack()`

- handles push and pop messages
- a clever way
- if we push something on the stack, should be able to push more things
- when we pop, we end the stack process
  - this allows push to send the most recently pushed value
  - don't forget to recall stack to be able to push and pop again!

```erl
stack(P) ->
  receive
    V   -> stack(P),
                P!V,
                stack(P);
    pop -> ok
  end.
```

Note: in this implementation, we assume we always have another process `P` to which we send all the stack values

- this implementation of a stack is a great stack, but we need to generate blanks if we reach the bottom of the stack
- to fix this, we can add on a small function

```erl
blank_stack(P) -> stack(P),
                  P!blank,
                  blank_stack(P).
```



