# Project ideas



## Certify clausal proofs produced by theory\_finite\_set. 

[Z3 exposes a method to log clausal proofs](https://microsoft.github.io/z3guide/programming/Proof%20Logs)

The idea is to run z3 to generate clausal proofs and validate theory axioms from the theory of finite sets.

* Whenever the theory creates a clause it has an option to supply additional parameters that can be used as proof hints.
 [The additional options are parameters that can be added to a clause](https://github.com/Z3Prover/z3/blob/62ee7ccf65d51c304553def478731aa17b848169/src/smt/smt_context.h#L970). An example where parameters are used in in the arithmetic solvers where they supply coefficients that can be used to certify validity of a disjunction of arithmetic inequalities.
 [Multiply the negation of each literal in the clause by the correspoing coefficient, add the inequalities up and you get a contradiction](https://github.com/Z3Prover/z3/blob/62ee7ccf65d51c304553def478731aa17b848169/src/smt/theory_arith_core.h#L1100).
 
* The OnClause functionality can be used to monitor theory axioms created during search. You can validate theory axioms from the finite\_set theory.

An example Python script that logs and prints proofs:


```
from z3 import *

s = SimpleSolver()
s.from_file("../f-proof.smt2")

def on_clause(p, deps, clause):
    print(p, deps, clause)

OnClause(s, on_clause)
print(s.check())

```
It prints:
```
asserted(s == set.union(t, u)) [] [s == set.union(t, u)]
asserted(set.in(x, t)) [] [set.in(x, t)]
asserted(Not(set.in(x, s))) [] [Not(set.in(x, s))]
finite-set(in-union) [] [Not(set.in(x, t)), set.in(x, set.union(t, u))]
```

where f-proof.smt2 contains 
```

(declare-const s (FiniteSet Int))
(declare-const t (FiniteSet Int))
(declare-const u (FiniteSet Int))
(declare-const x Int)

(assert (= s (set.union t u)))
(assert (set.in x t))
(assert (not (set.in x s)))
(check-sat)
```

In other words, the parameter _p_ contains a proof hint.

## Add (a model-based) fuzzer to theory_finite_set.

Apply methodology of Fazekas and Biere and others to add a model-based fuzzer
for the theory solver.


## Add a Boolean Lattice Refutation Solver


Constraints that involve equations between sets can sometimes be decided faster than the default complete decision procedure.

This project is to invent and develop a satisfiability checker for Boolean lattice constraints.

Boolean lattice constraints are of the form:

$$
s \subseteq t, s \subset t, s = t, s \neq t, s \not\subseteq t, s \not\subset t.
$$



The initial property of Boolean lattices is that every pair $s, t$ have a least upper bound and greatest lower bound (lattice property), 

and the de Morgan rules for set operations. The refutation checker can check for just lattice properties.

For example


$$
(s_1 \subseteq t_1 \vee s_1 \subseteq u_1) \wedge t_1 \subseteq s_2 \wedge u_1 \subseteq s_2 \wedge \ldots u_{n-1} \subseteq s_n \wedge s_1 \not\subseteq s_n
$$


is unsat. 

Certifying unsat is possible using only properties that $\subseteq$ is a partial order.

## Completness for finite sets with select and ranges.

This project is pen/paper as opposed to coding.

The combination of select (filter) and ranges wasn't considered and the model construction we pursue here
is not used in the reference literature on sets with range constructs. A rigorous pen/paper proof for
what are the sufficient set of instantations and model construction can be highly valuable weeding out
mistaes that can be made at the coding level.


## LIA*

Integrate a LIA* solver directly with z3.
The python prototype from 2020 does not work with updates to z3.

## Multi-sets and Cardinality constraints.
Multi-set reasoning can handle cardinality constraints as an alternative to Venn-diagram decomposition.
The project is to integrate enough of LIA* and multi-set reasoning to support cardinality constraints.

## Port the theory_finite_set to the "new" SMT core.

Z3 has two SMT cores. The _new_ core resides [here](https://github.com/Z3Prover/z3/tree/master/src/sat/smt)
where an effort has been made to make defining theory solvers a bit more streamlined than the main (legacy)
SMT core. The _new_ core is not used by default because many tools are based on behaviors of the legacy core,
and furthermore the new core could use tuning. Nevertheless it can be more efficient that the legacy core.
The task is to create finite_set_solver in the new core that replicates the functionality of the solver
developed during class. The project presents an opportunity to read the solver concepts "by using your fingers".

* The new core also has better native support for built-in proof checking. It follows a plugin model for theory solvers to define certifiers. For example EUF is checked by a small module that understands union find, and replays congruence closure based on [proof hints](https://github.com/Z3Prover/z3/blob/62ee7ccf65d51c304553def478731aa17b848169/src/sat/smt/euf_proof_checker.cpp#L65).

## Solver tuning

The solver for the theory of arrays uses a set of pruning techniques
to avoid redundant axiom instantiations. The finite set solver uses
a queue of axioms and prioritized conflicts and unit propagations.
It has place-holders for setting weights of theory axioms.
How do the approaches compare? Can we learn from one approach
that the other does not address?

Implementation wise, there are several places that can be tuned.
It is better to vet areas for improvement with benchmarks.

## But there is a problem

Examine correctness/incorrectness of saturation over sets when the base sort is finite.
Are extensionality axioms sufficient to guarantee completeness for sets over finite base sorts?


## Finite Sets and local search

Supposed you are given a conjunction of finite set constraints.
Define a local search repair algorithm for finite sets.
A repair step can be to add or remove elements from a current
assignment to set variables. Constraints are evaluated based
on the assignment to each set variable.

Z3 contains local search plugins for several theories in the ast/sls directory.


## Quantifier solving for finite sets

We already know that weak monadic second-order logic is decidable.
So we can solve quantified formulas over finite sets in many cases.
Can you establish quantifier elimination or satisfiabilty modulo
quantifiers directly (and for additional operators)?

There is quantifier solving for Arrays in Z3.
It is documented in an FMCAD paper with Arie Gurfinkel et al.
Use this as a starting point.

## Experimental benchmarking

* Curate and collect benchmarks for finite set SMT problems.
* Set up evaluation framework and benchmark.
