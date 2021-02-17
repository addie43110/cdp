# Exercise Sheet 6

## Tuple server in Erlang
- this is the Linda Tuple server we implemented in lecture, but now with timeouts
- it should be possible for:
  - in requests with timeout
  - out requests with timeout: this means the tuple only stays for `timeout` time in the tuple space
  - rd requests with timeout

Solution:
- this is solved by sending the timeout information to the tuple server along with the request
- timeout information can be `{just, Timeout}` or `nothing`
- everytime the server gets a new tuple, it checks the requests and times of the requests to see if not only the new tuple matches the request, but also if the request is still valid
- similarly, every time the server gets a new `in` or `rd` request, it goes through its tuples and checks to see if they are valid or expired, and returns only ones which are valid
- helper functions `calc_max_time()` and `timedout()` calculate the time a tuple or request is timed out and check if a timeout is expired repectively

## Reading from a remote channel
- here, we just want to add reading a remote channel 
- so we add a `remote_read` option in the `portListener`
- and we add another `read_chan({remote_chan, Host, Port})` case, where we send the `remote_read` message to the portListener function
- then we await an answer, close the socket, and deserialize the answer
- now in the Ping Pong server, we can read the `pong` answer directly from a channel located on the server

## Linking channels
- here we wanted to link channels so that they send out messages to each other every, say, 2 seconds
  - if they don't receive a message back, then that channel is presumed dead
- in the sample solution, we actually don't even check for a response message
- if the connection (socket) is closed or broken and we receive an error message, then we presume that channel is dead
- to implement this, we just need to add a `ping` and `{link, RemoteChan}` to the list of things portListener can receive
  - we still have to be able to accept `ping`s in order to prove we are still alive
  - but, we don't need to send back any message
- when we want to link to another channel, we begin a link process (which is basically the ping process)
  - we also need to notify the other channel that it should link to us
  - we send a boolean of `true` or `false` to the `link_chan` function
  - that way we know whether we need to tell the other channel to link to us, or if we *are* the other channel and we need to link to another channel
- the `link_chan` function either way spawns the `link_check` process, which connects to the other channel and sends a ping every 2 seconds, then closes the connection
  - that is, it keeps opening a new connection and socket every 2 seconds
- if the connection is broken, it sends the message `exit` to the local chan

## Sieve of Eratosthenes with `gen_server`
- confusing, to say the least
- just remember `handle_call` does work on the server and sends a reply to the client
- `handle_cast` simply does work on the server without sending a reply
