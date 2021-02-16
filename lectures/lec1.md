# Lecture 1

## What is concurrency?
- Allowing multiple threads or processes to run at the same time. 
- What is a thread or a process? These are usually simple programs. In some languages, they share common data but this is not the case in all languages.
- multiple processes are executed at the same time

## What is the motivation for concurrency?
- when concurrency was invented, there wasn’t really an idea of having multiple users. 
  - instead, the focus was more on running multiple applications at the same time
  - the system should allow reactivity in every situation
    - would be terrible if a GUI could not run at the same time as some computation
    - a webserver should react on every request and should not block while older requests are being served

## What is the difference between parallel programming and concurrency?
- parallel programming is using multiple computers
  - generally the goal here is to make things faster or more efficient
- concurrency is running multiple threads on one computer

## What is a distributed system? Why use distribution?
- Frank Huch says “Systems are distributed because our applications are distributed (ie. the internet, webservers)”
  - don’t want to bring all the clients on your system to only one computer to work
  - you want to bring your information to everyone out there in the internet
  - eg. telephones: doesn’t make sense to make everyone come to one telephone everytime they want to make a call
- as a secondary point, we can consider scaling and reducing latency


## What is a scheduler?
- a scheduler switches between threads and distributes processor time to the threads
- there are two main kinds of scheduling: cooperative and preemptive
  - cooperative: process decides when to surrender control to another process (process has to pass control back to the scheduler)
    - cooperative scheduling is more lightweight because you don’t need a structure which schedules and decides how to do things
  - preemptive: the scheduler decides and stops execution of processes
    - preemptive is nicer for the programmer because the programmer doesn’t have to worry about when to pass control to other programs
    - however, the overhead that comes with preemptive processing is becoming less and less important as computers evolve
- you might want to assign processes higher priority, but this is usually quite dangerous. it is better to weight all processes equally.

## `time.py`

Idea: create a timer of clock which contains hours and minutes. Use two threads to increment the minute field to show that without locking, we encounter strange behaviour.

Points:

- for small numbers of ticking, this program appears to work
- however, if we increase the number of minutes we tick, we begin to see an incorrect time.
- explore what happens when both threads read that the minute is 58
  - then the minute gets incremented twice to 60
- in our example, the hour and minute fields are **shared resources**
  - the code in the `tick()` method is the **critical section**
  - the to string method is also a critical section
  - critical sections are any sections which access the shared resource(s)

|**Function**|**Description**|
|---|---|
|`tick()`| increases the minute by 1 (and hour, if needed)|
|`with_leading_zero(n)`|  formats numbers with leading zeros for pretty printing|

### Code for threading in Python
```Python
import threading
# can also do this:
# from threading import Thread, Lock, RLock

class Time:
  def __init__(self):
    self.lock = threading.Lock()
    self.lock.acquire()
    self.lock.release()
    with self.lock:
      ...

def tick_n_times(time,n):
  ...

time = Time()

# if only one argument, write args=(time,)
t1 = threading.Thread(target=tick_n_times,args=(time,20))
t1.start() # don't forget to start the thread
t1.join()
```


