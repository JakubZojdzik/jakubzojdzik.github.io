---
title: 'Krivine machine'
summary: 'Explanation of Kirvine abstract machines'
date: 2026-04-21T18:50:00+02:00
draft: false
---

# Abstract machine

Abstract machine is a relation which defines transitions between states, which in our case allows to evaluate terms in lambda calculus.
It is useful, because it shows the exact implementation instead of just a set of rules.

# Evaluation strategy

The term in lambda calculus can be reduced in multiple ways. The reduction strategy is defined by the chosen order of reduction steps applied to term.

The call-by-name strategy is obtained by doing substitution before reducing argument in application.
This way, arguments are not evaluated if not needed. Such evaluation strategy is used by lazy languages.

On the other hand, the call-by-value strategy is obtained by restricting the beta rule (application) to value arguments.
It means, that before the substitution is done, the argument needs to be evaluated first.
Such strategy is used in strict languages.

The Kirvine machine implements call-by-name strategy

# Resulting term form

We define beta redex as a term of the following form:

\[
(\lambda x.A) M
\]

It can be reduced with beta-rule:

\[
(\lambda x.A) M \longrightarrow A[x := M]
\]

A term is in normal form if it contains no β-redexes.

Krivine machine doesn't reduce terms that much. Instead, it leaves terms in Weak Head Normal Form. It is the form to which functional languages evaluate.
It means, that it doesn't "look" into the lambda abstractions - function bodies can contain redexes. Also, we it doesn't reduce argument.

We only consider the head of a term. A WHNF is either:
- a lambda abstraction, or
- an application whose head is a variable

The syntax of WHNF can be described as:

\[
t ::= \lambda x.e \mid x e_{1} e_{2} \dots e_{n} 
\]

# Closure


The substitution used in β-reduction may duplicate terms and cause exponential growth during evaluation. To avoid this, the Krivine machine uses closures instead of direct substitution.
A closure is a term paired with its environment:

\[
c ::= \langle t, \rho \rangle
\]

An environment is a finite mapping from variables to closures:

\[
\rho ::= [] \mid \rho[x \mapsto c]
\]

I write $[]$ for the empty environment, and $\rho[x \mapsto c]$ for the environment extended with a binding of $x$ to closure $c$.

# Krivine machine

The state of Krivine machine is a triple

\[
(M, \rho, S)
\]

where:
- $M$ is a term,
- $\rho$ is an environment,
- $S$ is a stack of closures.

The machine is defined by the following transition rules:

\[
(\lambda x. M, \rho, c \cdot S) \longrightarrow (M, \rho[x \mapsto c], S)
\]

\[
(M\ N, \rho, S) \longrightarrow (M, \rho, ( \langle N, \rho \rangle ) \cdot S)
\]

\[
(x, \rho, S) \longrightarrow (M, \rho', S)
\quad \text{where } \rho(x) = \langle M, \rho' \rangle
\]

The machine starts from an initial state:

\[
(M, [], \bullet)
\]

where $M$ is a closed term.

A computation terminates when no rule applies. In the standard presentation, this corresponds to reaching a Weak Head Normal Form.
