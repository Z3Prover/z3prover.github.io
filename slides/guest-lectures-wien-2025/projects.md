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

The project can consider several levels of effort and difficulty:

* For every proof rule that uses an annotation that is supported write python (or other language supported by the API) code that matches the rule application.
* Use uninterpreted functions: translate theorems over finite sets to use uninterpreted functions. Axiomatize the theory using first-order axioms and validate the lemmas using Z3's quantifier instantiation engine (or Vampire or ..).
* Use uninterpreted functions and instantiate theory axioms directly. Use ground reasoning to validate theory lemmas.
* Develop a theory of finite sets using an interactive theory prover and derive a proof checker by either replaying the theory lemmas using the interactive theorem prover's automation or extract a certified checker that checks axiom instantiations.
* Develop a proof checker methodology for the inference rules used for set.size. In this case, the theory solver uses global reasoning to construct semi-linear sets for set.size expressions. It is much less direct to develop a proof checker for set.size reasoning, let alone a methodology. A successful project can develop the methodology alone.


## Add (a model-based) fuzzer to theory_finite_set.

Apply model-based fuzzers
for the theory solver.

* [Model-Based testing for SMT solvers](https://boolector.github.io/papers/NiemetzPreinerBiere-SMT17.pdf)

* [Model-Based Testing for Verification Back-Ends](https://www.semanticscholar.org/paper/Model-Based-Testing-for-Verification-Back-Ends-Artho-Biere/9bc19a853b5886ef22a956ed35d03abfa996fba0)

* Katalin has also used and developed Model-Based fuzzers.


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

### Using small models

You can also consider that the theory has a small model property: Set constraints are satisfiable if and only if they are satisfiable in a _small_ model.

* Given a bound $N$ on model size allocate $N$ bits for each set variable and check satisfiability of the Boolean encoding of the set constraints. For example, if $s \subseteq t$
 is asserted and $bv_s, bv_t$ are two bit-vectors of length $N$, then the bit-vector constraint $bv_t \& \~ bv_s = 0$ encodes the subset relation.
 
* Considering that the bound on the _small_ model may not be that small, does it make sense to add bits on demand?

## Completness for finite sets with filter and ranges.

This project is pen/paper as opposed to coding.

The combination of filter and ranges wasn't considered and the model construction we pursue here
is not used in the reference literature on sets with range constructs. A rigorous pen/paper proof for
what are the sufficient set of instantations and model construction can be highly valuable weeding out
mistaes that can be made at the coding level.

## Model-checker for conclusive answers for incomplete domains

There are several combinations of features that the solver (implementation) is unlikely to be complete for:

* set.range and set.size
* set.filter and set.size
* set.map
* set.range and set.filter

The base-line implementation (should) give up if there is a combination of operators that are not handled.
We would like to produce a conclusive SAT answer if the solver was able to produce a model.
Similarly, if the current state is satisfiable, there is no need for the solver to apply useless inferences.

Model construction produces an interpretation for every set variable. It is invoked if the state is satisfiable.
Nothing prevents us from invoking model construction even if we don't know whether the state is satisfiable.
We can use a candidate model for set variables and check if it satisfies all constraints (equalities, diseqalities,
does not create new equalities over shared variables, and values for set.size).


## LIA*

Integrate a LIA* solver directly with z3.
The python prototype from 2020 does not work with updates to z3.

This is not an easy project. The working version [theory_finite_set_size.cpp](https://github.com/Z3Prover/z3/blob/finite-sets/src/smt/theory_finite_set_size.cpp) contains a base implementation for checking set.size constraints. It suggests to use a different method than a general LIA* solver. One strategy is to extract conflicts by quering the arithmetic solver with a partially instantiated semi-linear set solution for cardinalities. These conflicts can prune the search space for propositional models of the solver that produces basis vectors.

## Multi-sets and Cardinality constraints.
Multi-set reasoning can handle cardinality constraints as an alternative to Venn-diagram decomposition.
The project is to integrate enough of LIA* and multi-set reasoning to support cardinality constraints.
This one is even tougher than the previous project idea. It appears not suitable for a 50 hour project, but if this is something for you, it is definitely fun.


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

## Quantifier solving for UFLIA

Z3 falls back to Model-based quantifier instantiation for UFLIA/UFLRA
The base solver for MBQI understands very little LIA. It is therefore unable to solve LIA formulas.
The QSAT procedure understands only LIA. It uses model-based projection (Cooper's method/Omega test)
to partially eliminate variables. It is possible to do better. The solver core enabled by setting sat.smt=true
uses a modification of MBQI that understands some amount of quantifier projection. It can solve problems
not in scope of basic MBQI and not in scope of QSAT.

A primary objective of this project is to understand the design space of solving UFLIA/UFLRA.

* What is state-of-the-art for solving UFLIA/UFLRA compared between solvers?
* What methods are essential for solving harder benchmarks?
