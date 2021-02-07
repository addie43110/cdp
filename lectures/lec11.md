# Lecture 11

## Implementing Erlang messaging using `gen_tcp`
- instead of sending messages directly to processes, we will send it to a `Chan`, and processes can read from the Channel
  - p1 sends message to channel c1
  - p2 reads messages from c1
  - we will also implement such that p1 can also read messages from c1 and see if that's a good idea or not
- thus, we need a remote representation of a channel that we can send across `tcp`

## Step 1: remaking channel in Erlang: `remote_chan.erl`
Idea: re-implement a channel (with remote possibilities) in Erlang

Points:
- instead of using MVars, a channel is now just a process
  - instead of keeping the state in a list, we will use a queue
  - the easiest way to implement a queue is just by using a mailbox

```erl
chan() ->
  receive
    {read,P} -> receive
                  {write,V} -> P!{chan_value,V},
                               chan()
                end
  end.
```

- in order to read something, we must have something written inside already
  - otherwise, we block (suspend)
- with this basic implementation, we have local chans
  - but we want to include the possibility of using remote


|**Function**|**Description**|
|---|---|
|`chan_register(Host, Name, Chan)`|connect to port 650002|

- in order to register chans, we can use our store program
  - we must store the name and a remote representation of the channel
  - the remote representation will be a port on which the tcp handler is listening and will forward the incoming messages to the channel
  - we will run on the fixed port 65002

`chan_register(Host,Name,Chan)`

- connect to port 65002, where the store is listening
- receive a socket back from the connection
- then we send `"store,<name>,<remote_chan>"` to the store, so they store a name associated with a remote chan representation
- the chan we are sending is still in a local form
  - we will change it to remote form using `local_to_remote(Chan)`
  - don't forget to send it as a string as well!

`chan_lookup(Host,Name)`

- connect to the store port
- send a lookup command
- `gen_tcp:recv(Sock,0)` to receive the string representation of the remote chan
- need to deserialize this chan string value!


`new_chan()`
- use `gen_tcp:listen(0,[args])`
  - note that passing 0 here means `gen_tcp` will pick any random available port
  - then it will listen on some free port
  - because the port acts as the idenfier for each channel, we need a new free port every time we make a new channel
- we can extract the port of the LSock using

```erl
{ok,LSock} = gen_tcp:listen(0, [list,{packet,line},{reuseaddr,true}]),
{ok,Port} = inet:port(LSock)
```

- then we create a local representation of a channel in a tuple containing the atom `local_chan`, the pid of the process (received from spawning `chan()`, the IP address, and the port number
- then we spawn a `portListener` which listens on LSock and connects to our channel, `Chan`

`portListener(LSock, Chan)`
- because we did not specific `active, false`, messages will come into our mailbox (actually not sure about this)
- so we receive: `{tcp,Sock,MsgStr}`
  - we convert the `MsgStr` into an Erlang value (by deserialization)
- then we write the message to directly to `Chan`
- port listener of course then loops

### local chan
```erl
{local_chan, Pid, IPaddr, Port}
```

### remote chan
```erl
{remote_chan, IPaddr, Port}
```

`write_chan()`
- if writing to a local chan, we know the pid so just directly send the message
- if writing to a remote chan, we must connect to the host and port
  - we will receive a socket back
  - send a message over the socket
  - don't forget to first `serialize()` the message! (this acts mostly like `base:show()`, but also will turn local chans into remote chans automatically, so no local chan pid gets sent accidentally)

`local_to_remote(Chan)`
- takes a local chan and turns it into a remote chan
- not complicated, just change the tuple
  
## Simple Ping Pong server in Erlang
Idea: show how to use our `remote_chan`

- we will have a server
  -  ping server listening on a certain port, will need to register it
- will also have a function `ping()` used to ping a server
- we will use our `remote_chan` implementation


|**Function**|**Description**|
|---|---|
|`start_ping()`| establish a channel on which our ping server listens, register the channel under the name `"ping"`, then enter the `ping_loop()`|


`ping()`

- lookup the ping channel, if it is running, continued below
- create an answer chan,
- then we will `write_chan` to the ping channel with the atom `ping` and the answer chan
  - don't forget to convert the answer channel to a remote channel
  - also change it to a string!
  - Frank solves this by writing a separate `serialize` function which automatically serializes any string passed to `write_chan`
- then read the answer chan to see if a `pong` has come back to us


`ping_loop()`

- recursively reads the ping channel and if there is a `ping` in there, read the answer channel and send a `pong` to the answer channel
