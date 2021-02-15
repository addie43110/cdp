# Lecture 25

## Transactions in a distributed setting
- we have three nodes
  - node 1 oversees the transaction and uses nodes on node 2, node 3, etc.
- we need to lock all variables in the verify and commit stage
  - but if we are not careful, can run into deadlock problem if two transactions lock in the opposite order
- two solutions:
  - global order
    - this implies we need to have an ordering for TVars
      - this will be defined by IP-addr and a TVar ID
    - locking must also be done sequentially
  - layback resources when lock is not available
    - here we can send a parallel lock message to all nodes
    - there are two possible answers: locked or failed
    - we can send a coordinator to collect all answers
    - if any lock returns `failed`, we can inform all the locked locks so that they free
    - then we can try locking again (ie. rollback the commit)
    - because we can lock in parallel, it will be faster

### Lay back locks approach
- when we send the lock message, we can add further information about the transaction
  - we can add the version of the TVar that we currently have
  - the TVar can send back the locked message
  - it checks if the version is the newest one, and sends back that confirmation
  - `{locked}` or maybe just sends back `ready`
  - if we fail because of lock, send `cannot get lock` because then we can immediately try committing again (only rollback commit)
  - if fail because `version invalid`, we need to rollback the entire transaction
- the phase of locking and distributing information is combined
  - it is called a PREPARE action or message in the two-phase-commit protocol
- we have a two parties: a coordinator and agents (which are the TVars on the nodes)
- the coordinator send the PREPARE message (sends the version and the WS-part appropriate for the TVar)
- the agents send back either a READY message or a FAILED message
  - in our case, ready means we are locked and the version is still valid
- if all the agents reply READY, we can enter Phase 2

#### Phase 2
- coordinator sends message to agents that all agents should commit, COMMIT
  - when the agent receives a commit, it commits the WS-part it contains and frees the locks
- coordinator awaits response from agents to confirm that all have committed, ACK
  - if any agent(s) reply FAILED, the coordinator sends ABORT to all agents
    - when the agent receives ABORT, it does not modify the WS-Part TVars and frees the locks
  - agents can send back an ACK, but it might not really be necessary


### Are there problems with the above approach?
- problems occur in distributed systems when nodes crash or connections are lost
  - this method is heavily dependent on the coordinator being available
  - if the coordinator crashes, the rest f the nodes are lost

Consider if an agent crashes:

- if an agent crashes after the PREPARE message is sent
  - coordinator simply won't receive any READY/FAILED message from the crashed agent
  - this is solved by adding a timeout on the coordinator while it waits for the agent's response
- if the agent crashes after the COMMIT message
  - here we assume the agent crashes and never comes back again
  - it's a much bigger problem is the agent loses connection then comes back again later with outdated information, but that's out of scope for this course
  - here, the agent crashes and the information is lost on that node forever
  - no ACK will be sent back
  - again, we can solve this by using a timeout
- if we crash after the ABORT message,
  - then again use a timeout
  - in this stage, the ACK doesn't even need to be sent, so this is not so critical

Thus:

- if an agent crashes, the coordinator can use timeouts to finish the protocol without producing an inconsistent global state

Consider if the coordinator crashes:

- if the coordinator crashes while sending PREPARE messages
  - some PREPARE messages get sent but not all of them
  - now some agents are already locked and are awaiting the COMMIT/ABORT message
  - here, we can't just set a timeout because we have no idea how long it takes the coordinator to get all the locks, for example.
- if the coordinator crashes while sending COMMIT messages
  - now some of the agents have already commited their WS and others are still waiting
  - some locks are still held
- if we crash during the abort stage, some locks are released, others are still held

Thus:

- blocking of the system is possible if the coordinator crashes
- also part of the transaction may end up committed
  - however, an inconsistent view is avoided (by blocking)
- one solution is to try to find another coordinator if an agent detects that the original coordinator is not responding (via timeout)
  - we can send all the information the coordinator knows with the PREPARE message
  - then the agents can get together an elect a new leader (leader election process), but how they do this is out of scope for this course
- another solution is called the three-phase-commit protocol
  - this doesn't actually solve all the problems, but some of them
  - instead of sending a PREPARE message follwed by a COMMIT message, we send some additional messages in between
  - we split the COMMIT message into two messages
  - coordinator sends PREPARE; agents send READY/FAILED
  - coordinator sends PRECOMMIT message; agents send PC-ACK
  - coordinator sends COMMIT; agents send ACK

Three-phase-commit notes:

- if coord crashes during PREPARE, still a problem
- if coord crashes during PRECOMMIT, agent can detect the problem using a timeout because the agent knows that the precommit should not take a long time (there is no locking involved, so no waiting on unavailable locks)
  - agents can discuss whether the precommit was received by all and if they should go forward with waiting for the commit

Note: commiting in a distributed setting is closely related to the problem of transaction comm distributed database systems. This is independent of using optimmistic or pessimistic transaction implementations.

The approach of direct notification of making other transactions invalid cannot easily be transferred to a distributed setting because of delays in network communication.
