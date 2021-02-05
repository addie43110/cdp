# Lecture 8

## Distributed applications in Erlang
- we know we can send messages easily between processes in Erlang
  - we can also send messages to processes on different machines
- if you send a message to a process on the same node / computer, it is quicker
  - you don't have to copy the message because of immutibaility
  - data cannot be changed, so can simply send the reference to the original data
- if you send it to another machine, it must be converted to a string and passed via the internet
  - here, we have to convert the message / data to string (byte array)
  - however, usually the messages we pass are quite small and can be sent in one package, so it's not so bad and often comes with multiple advantages

### How can we send a message to another process on a different machine?
- from an Erlang view, you only need the PID
  - pid contains all the information about its location on the internet


### Remote store server
- if we try to connect to the server from a different node, we need to know the server Pid
  - in order to know this, we need to register the store
  - registering maps a service name to a PID
    - a service name is simply a string
- in erlang, registry is ubiquitous
  - registry is located on every node running on the internet
  - ie. can register a pid in russia and look it up in germany
- is it necessary to register the client processes?
  - no, it is not. when we send a lookup message, we can send our own client pid with it

### Registration in Erlang
```erl
Server_pid = spawn(fun() -> store([]) end),
register(store_server,Server_pid).

% successful registration returns true

% Server_pid is the PID
% store_server is the string/atom
```

### To run a node in Erlang
```erl
erl -sname <name>
```

### To send messages to processes on other nodes
```erl
{<name>, server@server} ! {msg}.

% concrete example
{store_server, server@osdorf} ! {lookup, self()}.
```

## `chat_server.erl`
Points:
- needs a list of connected clients
  - should also have the names of the clients too, not only the pids
- be able to send a message to all clients
- process a client login
  - need the client pid and client name
  - tell other clients someone new logged in
  - tell the client they successfully logged in
- process a client logout

## `chat_client.erl`
Points:
- we join to a server node, `join(Node,Name)`
  - `Node` is the server node name, eg. `server@osdorf`
  - `Name` is our client name, eg. `Frank`
- in a concurrent setting, timeouts are usually to be avoided, but in a distributed setting, timeouts are helpful
  - for example, we should timeout after 2 seconds if the client still hasn't received a logged in message from the server
  - maybe the server is down or we made a typo or something
- printing done on the client-side allows for different clients to print the same thing different ways

Lecture ends on a cliffhanger: we send the wrong pid in the `user_input` function to the server.
