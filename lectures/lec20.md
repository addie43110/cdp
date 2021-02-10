# Lecture 20

## `stm.erl` continued

- we are fixing up our `retry` case code
  - right now, we receive too many `restart` messages in our mailboxes
  - additionally, we do not `unsusp` ourselves from linked TVars


### `restart_forward(Pid)`
- we spawn a new process `restart_forward()` with our own pid as a parameter
- we then enroll the `restart_forward` pid in all the TVars (instead of our pid, as was originally)
- then when the TVars change, they will notify our `restart_forward` process
- as soon as `restart_forward` receives one message from a TVar, it immediately terminates
  - the other messages are then lost to the void
- `restart_forward` sends our pid the `restart` message before it terminates, ensuring that we only get one `restart` message even when multiple TVars we were watching are changed

### Using TVars with versions
- motivation: right now we check the value of the TVar and see if it's the same as the one we have in our readset
  - if the values stored in the TVars are very large, this can take a lot longer
- one way to fix this problem is by saying that each TVar has a version number which gets updated each time the TVar is written to
  - it can be the case the the TVar gets the same value as previous written into it and in that case, since we check the version number, we will still rollback (as the version numbers won't match)
  - yes, this is an unnecessary rollback, but it can still be a better method than checking TVar values, especially when those values are large / complex

### `orelse`
- if the execution of T1 succeeds, then this is the result
- if the execution of T1 runs into a valid retry, then T2 is executed

Which read and write sets do we use if we run into a retry?
- say we have RS0 and WS0 before executing T1
- we execute T1 and run into a retry. Now we have RS1 and WS1 
- we must execute T2 now, and we will use WS0 because we do not commit the values of T1
-  however, we must execute T2 with RS1 because RS1 is what got us into the retry first of all
- reaching the retry, and therefore T2, is only valid based on the RS which lead to the retry
  - if it's not valid in the end, then maybe running into the retry was not valid either- also if T2 runs into a rollback, we must rollback the entire `orelse` since T1 could have changed as well

```erl
orelse(T1,T2) ->
  WS0 = get(ws),
  case catch(T1()) of
    rollback -> throw(rollback);
    retry    -> put(ws,WS0)
                T2();
    Res      -> Res
  end.
```


