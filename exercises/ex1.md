# Exercise Sheet 1

## Counter (in Python)
- the point of the exercise is to create a counter which prints the following output:

```
counter1: 1
counter1: 2
counter1: 3
```

- this counter has a constructor which takes the waiting time in `msec` and a `name`
- part two of this exercise was to be able to start multiple counters from the command line using the command:

```py
python counter.py 300 500 1000
```

Points: 
- the exercise, i think, was just to get us used to working with threads in python
- no need for any locking or sharing of resources in this exercise
- we can create a new thread for each instance of the class by passing `threading.Thread` in the class constructor

```py
import threading

class Counter(threading.Thread):
  def __init__(self,msec,name):
    threading.Thread.__init__(self)
    self.msec = msec
    self.name = name
    self.value = 0
```

## Dining Philosophers
- implement the dining philosopher's in python using the method of laying back locks
- answer why it is not necessary to perform the two actions (checking if the stick is available and taking it) as one atomic action
  - in the worst case, we follow the philsophers around in a circle and the last philosopher will see that the second stick is already taken, so he will put his sticks down
  - thus, the second to last philsopher is able to take the stick and eat
  - following suit, all the rest of the philsophers are able to eat as well, back down to the first


## Extensions for RLock class
- we implemented the RLock in python in class
- in this exercise, we needed to add `locked()` and `tryAcquire()` to that implementation
- for `locked()`, we used a regular lock in the implementation, so you could just return `self.lock.locked()` for that (seems like cheating to me)
- again, for `tryAcquire()`, we just used our own lock but with `blocking=False` when our own blocking was set to false
  - `if self.lock.acquire(blocking)`

- for part 2, we had to implement the RLock class in Java
- we were only allowed to use Java's Semaphores to help us
- I think this is fairly straight forward:
  - we use a `counter`, `holdingThread` and Seamphore `lock` as our class attributes and implement pretty much the same as we did in Python


## Implementation of semaphores
- we had to reimplement `Semaphore` in Python
- only had to implement `acquire` and `release` without blocking or timeouts or anything like that
- I think our solution was more elegant that the given solution
- we used a Condition variable
  - on `acquire`, while `len(self.holding_threads) >= self.limit)`, we simply waited (`self.cond.wait()`)
  - we kept track of all the holding threads in a list
  - in the `release`, we simply checked if the thread wanting to release the semaphore had actually taken it
  - if it had, we removed it from the list of holding threads and notified any sleeping threads in the `acquire`

- for part 2, we were to make a test program which creates a large number of threads but only 3 are allowed to enter any given critical section
- furthermore, our program needed to be able to tell if/when there were more than 3 threads in the critical section
- again, this was in Python

- for our implementation, we made the Test class track the number of threads in the critical section
- the Test class had `sem` and `num_in_crit` attributes
- the critical section was a function which acquired the `sem` and then incrememented the number of threads in the critical section, `num_in_crit`
- then it slept, and once it woke up, it decreased the number of threads in the critical section and released the semaphor
- note that this Test class also still had a regular lock `lock` to make sure the incrementing and decrementing of the `num_in_crit` remained synchronized
