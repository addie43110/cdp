# Exercise Sheet 4

## Extension of Server-based Chat
- the server based chat as discussed in Lectures, but with fun stuff like:
  - names of participants must be unique
  - pretty print list of participants
  - enable private messages
  - command to show list of participants
- pretty straight forward

## Serverless Chat
- create a chat where you join by knowing a chat member
- each chat is registered under the name `chat`, just on different nodes
- consists largely of three stages:
  - connect to an existing chat member
  - they will send back the list of clients the know of
  - we then start the chat
    - set up everything including linking ourselves to every client in the client list
    - we register ourselves under `chat`
    - we broadcast that we have joined the chat
    - we spawn the IO input process and then move to the main chat server
  - chat server receives messages from other clients and the IO input process

Problem:
- the problem with this implementation is if two clients join at the same time at different nodes, they will not know of each other
- to fix this, we can always sort the list of clients and then force newcomers to join via the leader

## Distributed game in Erlang
- this game is just clicking a button which appears randomly on screen
- players should be able to join and exit the game without affecting other players
- while fun, i am not so sure creating this game gave us any more insights to distributed programming in Erlang that the chat program didn't already cover
