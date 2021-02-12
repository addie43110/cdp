# Exercise Sheet 3

## Messages in Erlang
- here we had to write a small test program which illustrated the matching order of the `receive` expression
  - the mailbox acts like a queue, so messages are received in the order they are sent
- we then had to implement a counter process (server) which can be incremented by client processes

## Counter in Erlang
- this time, we had to implement the CounterGUI in Erlang
- this turned out to be much simpler than as implemented in Python because we only had to fill in the message receiving part in the server code
- there were two processes associated with each "Counter"
  - the counter server which handled requests
  - a ticker method which sent value updates to the counter every so often milliseconds
  - this meant each counter had to remember the ticker Pid
  - furthermore, each ticker needed to know the pid of the counter
  - lastly, if the counter was stopped, the ticker needed to be stopped as well: `exit(TPid, ok)`
  - if the counter was resumed, a new Ticker had to be made and it needed to start with the value it left off on

## Sieve of Eratoshenes
- model the sieve as a mesh of Erlang processes
- each sieve process requests a potential prime number by its predecessor
- it filters the multiples of "its" prime number and sends the next possible prime numbers to its successor
  - to stop processes from generating an infinite number of possible prime numbers, the implementation should be demand-driven
  - sieve only sends the next number when asked for

Musterlösung:
- `allNumbers(N)` process just generates the set of natural numbers
- `sieve(Pid)` asks the `Pid` for the next number, then waits for a collector pid to send a message
  - `sieve` then sends the collector pid the current number AND the child of the `sieve` (which has the `sieve` pid as its pid)
- a lot more functions to filter primes, collect them, and then collect the firstNPrimes
- personally I found this approach very confusing
- my approach was definitely not better, but at least i understand it
- we start with an initial generator (similar to the `all_numbers(N)`
- then we call the `sieve` processes
  - sieve keeps track of `PredPid`, `SieveNum`, `Primes`, `Total`, and `Waiting`
  - when a sieve is born, the first number it receives from it's predicessor (`PredPid`) has to be a prime
  - `Total` is used to keep track of how many times the sieve has asked `PredPid` for a number
  - the second number the sieve receives from `PredPid` will become the filter or `SieveNum` for a child sieve process
  - other than that, the sieve filters out numbers when `Num rem SieveNum == 0`
- I had `Primes` and `Waiting` to try to keep track of waiting Pids, but I don't think this was necessary
- each sieve process should have exactly one parent process and one child process, so there should never be a case where the sieve needs to send the same number to multiple child processes or have multiple child processes waiting on it


## Semaphores in Erlang
- implement semaphores in Erlang with the following functions:
  - `p/1` (acquire)
  - `v/1` (release)
  - `l/1`: returns the number of threads still allowed to acquire the semaphore
- the sample solution is quite short and assumes proper client usage of the semaphore (ie: no `release` before an `acquire`)
- our solution is unnecessarily complicated, but definitely more guarded as it keeps track of holding threads
  - our solution also keeps track of waiting pids, but in hindsight, I don't think this is necessary
