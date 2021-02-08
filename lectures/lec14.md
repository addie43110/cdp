# Lecture 14

## Linear Time Logic (LTL) (Linear Temporal Logic??)
- a program can be an infinite series of states
  - in a concurrent system, this series of states can branch
  - branching because we have multiple processes and we don't know which one will execute first or in which order
  - thus we can enter different steps
  - we can obtain different results depending on who gets scheduled first 

- within each state, certain properties hold or don't hold
- we want to express a formula that holds over the whole path

### Syntax

LTL formula is:
- true
- false
- a property, P
- not Psi, where Psi is a LTL formula
- Phi and Psi, if Phi and Psi are LTL formulas
- Phi or Psi, if Phi and Psi are LTL formulas
- X(Phi), where X is called 'next' and Phi is a LTL formula
- Phi U Psi, where U is called 'until' and Phi and Psi are LTL formulas
  - Phi holds until some step and on the NEXT step, Psi must hold

Abbreviations:
- finally, F(Phi) = true U Phi
- globally, G(Phi) = not F not Phi
- infinitely often Phi holds, F\_infinity (Phi) = G(F(Phi))
- Phi holds alwys with finitely many exceptions, G\_infinity (Phi) = F(G(Phi))

### Semantics
- consider infinite paths
- difficult to type, see notes


### Equivalences on LTL
- F(X(Phi)) is equivalent to X(F(Phi))
- De Morgan's laws also apply for negation on conjunction and disjunction
- not (not Phi) = Phi
- not(X(Phi)) = X(not(Phi))
- not(F(Phi)) = G(not(Phi))
- not(G(Phi)) = F(not(Phi))
- not(Phi U Psi) = (not Phi) R (not Psi)

Shifting remaining decision to next state:

- F(Phi) = Phi or X(F(Phi))
- G(Phi) = Phi and X(G(Phi))
- Phi U Psi = Psi or (Phi and X(Phi U Psi))
- Phi R Psi = Psi and (Phi or X(Phi R Psi))

Generalizations:
- finally usually used in liveness properties
- globally usually used for safety properties
- finally infinitely and globally infinitely are usually used for fairness

## `ltl.erl`
- first, we just represent all the syntax as (nested) tuples
- `normalize` pushes all existing negation as far "in" as possible
  - don't forget we sill have to define it for non negated properties
