# Exercise Sheet 5

## Serverless chat
- This exercise took the idea from last exercise where two clients who join at the same time on different nodes have no knowledge of each other
- one of the suggestions was to send a hash of the client list with every message
  - thus, if you every received a message which hashcode didn't match your own, you knew you were out of sync with the clients somehow
  - this of course only works if someone whose client list is different than yours sends everyone/you a message
- a better solution was discussed in the exercises: establish a leader of the client list and force all logins to go through the leader
- when you first reach out to a chat client, they will send you their leader
- then you can connect via the leader
- we sort the list of clients to find the leader (don't forget to include yourself!)
- other than that, the code largely stays the same

## TCP-Store in Python
- implement a store-server application in Python using python's TCP library
- if you do this successfully, you should be able to use the Python server code with an Erlang client or vice versa

```py
# create an INET, STREAMing socket
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# bind socket to host and port
serversocket.bind(("localhost", 65002))
# listen on the socket
# the 5 here means we can queue up as many as 5
# connect requests before refusing outside connections
serversocket.listen(5)
```

- and here is the basic code for receiving a message over a socket

```py
def requestLoop(clientsocket):
  alive = True
  while alive:
    chunks = []
    while True:
      chunk = clientsocket.recv(1)
      if chunk == b'':
        alive = False
        break
      elif chunk.decode("utf-8") == "\n":
        break
      chunks.append(chunk)
    requestHandler(clientsocket, b''.join(chunks).decode("utf-8"))
```

- the `requestHandler` then parses the message and does stuff based on what the message was
- it can also send things back over the clientsocket as follows:

```py
clientsocket.send(("{just,"+v+"}\n").encode("utf-8"))
```

- lastly, we have the accept loop which just accepts connections to the serversocket

```py
while True:
  (clientsocket, address) = serversocket.accept()
  threading.Thread(target=requestLoop,args=[clientsocket]).start()
```

## TCP chat in Erlang
- reimplement the Server chat in Erlang using TCP (no message passing between nodes!)
- i think most of the TCP syntax is covered in the TCP lecture

## Extension of RemoteChan
- here we had to update the RemoteChan code we made in lecture
- Chans used locally should not have a port listener made
- a remotechan returning to its local node should be converted back into a localchan

Points:
- first of all, when making a new chan we always make a local_chan
- then we create a channel database where we can register remote channels
  - when we register channels, we send in the local_chan representation of them and the channel database creates a host and port for them, then saves them as a remote chan
- changing local channels into remote channels is now just a question of looking them up in the channel database
  - the Pid is the key
- to solve changing remote channels back into local channels, we check if the host is "localhost"
  - if so, we can look it up in our channel db (using the port this time) to get the Pid of the local chan
  - if we get the local chan, we can return the `{local_chan, Pid}`, otherwise we must keep it a `{remote_chan,Host,Port}`
- note that the channel db needs to support looking up channels by pids and ports
