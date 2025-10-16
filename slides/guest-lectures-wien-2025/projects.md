# Project ideas 

## Certification



Certify clausal proofs produced by theory\_finite\_set. 


* Whenever the theory creates a clause it has an option to supply additional parameters that can be used as proof hints.
* The OnClause functionality can be used to monitor theory axioms created during search. You can validate theory axioms from the finite\_set theory.



## Add a Boolean Lattice Refutation Checker



Constraints that involve equations between sets can sometimes be decided faster than the default complete decision procedure.

This project is to invent and develop a satisfiability checker for Boolean lattice constraints.

Boolean lattice constraints are of the form:

* $s \subseteq t, s \subset t, s = t, s \neq t, s \not\subseteq t, s \not\subset t$.



The initial property of Boolean lattices is that every pair $s, t$ have a least upper bound and greatest lower bound (lattice property), 

and the de Morgan rules for set operations. The refutation checker can check for just lattice properties.

For example

$(s_1 \subseteq t_1 \vee s_1 \subseteq u_1) \wedge t_1 \subseteq s_2 \wedge u_1 \subseteq s_2 \wedge \ldots u_{n-1} \subseteq s_n \wedge s_1 \not\subseteq s_n$ is unsat. 

Certifying unsat is possible using only properties that $\subseteq$ is a partial order.



## Multi-sets



