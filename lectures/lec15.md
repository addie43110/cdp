# Lecture 15

## `ltl.erl`

Recap

- we have make smart functions to make our formulas (which are just nested tuples)
- we have implemented `normalize()` to push negation (if any) inside as far as possible
- Huch adds `showLTL()` which is just used for pretty printing

`start()`

- starts our LTL process which allows us to check formulas while our application is running
- first list: formulas which must be checked at runtime
- second list: original formulas
- third list: propositions which hold in the current state
- we will also register this process so we can directly send messages to it without knowing its pid

`ltl(Phis,Props)`
- can `{assert,Phi}`: assert a new formula Phi
  - normalize `Phi` first
  - check if `Phi` holds in this state
    - if it returns true, formula is already satisfied so don't need to add it to `Phis`
    - if it fails, we print an assertion violation and do not add it to `Phis`
    - if checking gives us back a new formula `Phi1`, we should add `Phi1` to `Phis`

`{new_prop,P}`

- whenever the props are changed, this is considered a new state (or node on our path)
  - thus, we need to check if this prop is already in our prop set
  - if it is not, we need to check all our formulas again to see if they still hold
  - note: we need to analyse the checked formulas to see if any returned `true` or `false`, because then they can be thrown out and/or an assertion violation must be thrown

- `{release_prop,P}`

- similar has to be done in the release prop set

`assertion_violation(Phi)`

- print that the formula `Phi` was violated
- however `Phi` has been normalized many times so might look very different from how it was first entered
- thus, we keep track of the original formulas and print the original one instead

`check(Phi)`

- this needs to loop over all possible formulas an check if `Phi` can be reduced to `true`,`false`, or a new formula, `Phi1`
- watch out for conjunction and disjunction
  - `check(Phi)` and `check(Psi)` each have three possible return values, thus we need to manually implement `or` and `and`
- Huch keeps `x(Phi)` as `x(Phi)` in the check function, instead of returning `Phi`
  - this means we need to implement a step function which steps over any possible `x`
- note that the result of the check function are only `x`, `disj` and `conj` formulas

## `critical.erl`
Idea: an example application which uses our `ltl.erl` and shows how it works

- this is a simple store which only stores one number
  - the number can be looked up or set
- we want to increment and decrement the value
- the critical section is when we are modifying the value of the store
  - thus, we shouldn't be both incrementing and decrementing at the same time (in the critical sections at the same time)


## LTL Theory
- is it possible for a formula `f(Phi)` to be violated when tested?
  - we can definitely prove `f(Phi)`, but it cannot be violated
- similarly, `g(Phi)` can only be violated, but not proven

Consider the following scenario:

```erl
main() -> assert(f(prop(done))),
          some_complicated_computation(),
          new_prop(done).
```

This is essentially the halting problem. Thus, it is impossible to determine whether finally a formula holds or not after some complicated computation
