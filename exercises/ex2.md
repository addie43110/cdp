# Exercise Sheet 2

## Extension of MVar
- in Python, extend the MVar class with the following functions
  - `read()`: reads MVar without emptying it; blocks if MVar is empty
  - `try_put(v)`: returns boolean of success
  - `try_take()`: returns a Maybe value
  - `swap(v)`: replaces value in the MVar; blocks if MVar is empty

Points:
- in `read()`, while `self.empty`, we sleep
- we are awoken when a process `put`s something in the MVar
- however, consider the following situations:
  - a `take()` thread wants to take from an empty MVar, so falls asleep
  - a `read()` thread then comes along and since the MVar is empty, it also falls asleep
  - a `put()` thread then puts something in the MVar and notifies the `read()` thread to wake up
  - if the `read()` thread does not notify the `take()` thread to wake up, effectively 'a notify is lost' and we end up in a deadlock
  - this is because any `take`s will never awaken so the MVar will always be full
  - and since the MVar is always full, no `put()` threads can write to it either

- note also there is heavy reliance on the `self.empty` attribute to avoid deadlock
  - because this attribute can only be either `True` or `False`, we can avoid situations where both read and write locks lock in the opposite order on two different threads

- in our implementation of `take_timeout()` and `put_timeout(v, timeout)`, we simply used the parameter `timeout` as passed it to the condition variables

```py
notified = self.writeCond.wait(timeout)
if not notified:
  return False
```

- however, the sample solution actually starts a timer at the beginning of the function to make the timeout more accurate
- it does, however, pass the `remaining_timeout` to the condition variable, so it's really not that different
- the sample solution also throws an error `"timeout"` message instead of returning `False`
- this is actually a much better idea than our solution

```py
raise TimeoutError("timeout")
```

## Counter with GUI
- using TkInter, create a basic GUI for the counter class in Python
- every counter should be shown and controlled froma separate window
- each window should have the following buttons:
  - stop
  - start
  - close
  - copy
  - clone
- make sure the program terminates if all counters are closed

Points:
- in our implemenation, we used both an `end_flag` and an `_is_running` boolean
- thus, we could pause the counter by setting `_is_running` to false without have the counter terminate (`end_flag`)
- we used a condition variable so when the counter was paused, it would sleep and when the counter was resumed, the condition variable notified any sleeping threads
- we also kept a global lock (probably a bad idea) just so that we could keep count of how many windows were open
- then, if the number of windows ever reached 0, we destroyed the root of the program
- i think this stems from a problem we had when closing all the windows and counters failed to terminate the program
- looking at the sample solution, it is largely the same, so perhaps this was just a bug using outdated code in the TkInter package
- the sample solution did however have a `close()` funcion in the `CounterGUI` class which kept track of how many windows were open
  - by contrast, our solution used `window.destroy()` to close windows, so perhaps this was the root of the problem

## Extension of Chan
- take the lecture implementation of `Chan` in Python and extend it by adding the following methods
  - `add_multiple`
  - `is_empty`
  - `un_get`: adds an element to the "wrong" end of the `Chan`
- lastly, we had to implement making bounded size Chans

Points:
- to restrict the size of a Chan, we can simply add a semaphore
  - threads which wish the `write`, `un_get` and `add_multiple` must acquire the semaphore first
  - threads which read the Chan can release the semaphore
- `add_multiple` was implemented as just `for elem in elems: self.write(elem)`
- `is_empty` simply checks if the count is 0
- note in the first (naive) implementation of `is_empty`, if the chan is empty and another thread is waiting to read the chan, `is_empty` will suspend because the other thread is holding the read head MVar
  - although, the "naive" implemation reads both the read value and the write value and then checks if they are equal, which is pretty dang naive if you ask me
- a good way to fix this is to use a count variable which keeps track of how many elements are in the Chan at a given time


