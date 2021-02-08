# Lecture 13

## Back to remote chans....
- EXAM: implement the second possibility of remote channel
  - this is the one where there is only one socket which receives messages for all processes on a node
  - the repesentation of c2 on node 1 needs to contain an identifier for c2 on node 2, and the tcp_handler needs a mapping from ids to channels

### Comparison with RMI/RPC
- RMI uses remote objects
  - each remote object has attributes and methods
  - thus, if processes on node 1 know remote object o on node 2, it can simply call the method
    - `o.method1(x,remote_obj2)`
  - then node 2 can call methods on `remote_obj2`, similar to how we passed remote channels over TCP
  - we still need identifiers for objects
  - we could even serialize `remote_obj2` and make a copy directly on node 2
    - this can reduce network traffic when calling methods/attributes of `remote_obj2`
    - however, we produce a copy, so the state of the original `remote_obj2` cannot be changed from node2
  - we also still need to register the objects for first contact
- RMI/RPC extends sequential programming to a distributed setting
  - however, there is concurrency involved
  - RMI objects (eg. servers) can be accessed concurrently by multiple parallel clients
  - thus, you need to consider all concurrent programming problems (eg. critical sections, mutual exclusion, deadlocks) in addition to the distributed program problems

### How we can use linking to make our application more robust
- how can we implement linking using our remote channels?
- there are two possibilities
  - can link a process to a channel
  - or we could link two channels to each other
    - we are sending messages to tell the other process that we crashed anywas, so might as well use channels
    - idea: send crash message to the linked channel when we crash

Link two channels together

- we can create a link table which contains all links for a channel
- the problem is, if a node crashes is cannot send another message
  - to solve this, we will use the heartbeat technique
  - we ping each channel on our link table every so often and if we don't receive any reply after some time, we assume they are dead
  - thus, each channel needs a process running in the background (eg `linkchecker()`) which does that

## Erlang `gen_server`
- Learn You Some Erlang for Great Good!
- send a message to the server and wait for a reply
  - `handle_call()`
  - return value of the function is sent back to the client
- send a message to the server and don't wait for any reply
  - `handle_cast()`
- client methods are defined in the server code!
  - clients then use these handles to change, receive server information
   
### `gen_chat.erl`
Idea: the generic version of our chat

Also see: `gen_chat_client.erl`
