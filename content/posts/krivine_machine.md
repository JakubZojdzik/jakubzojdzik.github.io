---
title: 'Krivine machine'
summary: 'Explanation of the Krivine abstract machine'
date: 2026-04-21T18:50:00+02:00
draft: false
---

## Abstract machine

An abstract machine is a relation that defines transitions between states, which in our case allows us to evaluate terms in lambda calculus.
It is useful because it shows an operational implementation instead of only a set of reduction rules.

## Evaluation strategy

A lambda-calculus term can be reduced in multiple ways. A reduction strategy is defined by the order in which reduction steps are applied.

The call-by-name strategy performs substitution before reducing the argument in an application.
This way, arguments are not evaluated unless needed. This evaluation strategy is used by lazy languages.

On the other hand, call-by-value is obtained by restricting the beta rule (application) to value arguments.
This means that before substitution is performed, the argument must be evaluated first.
This strategy is used in strict languages.

The Krivine machine implements the call-by-name strategy.

## Resulting term form

We define a beta-redex as a term of the following form:

$$
(\lambda x.A) M
$$

It can be reduced using the beta rule:

$$
(\lambda x.A) M \longrightarrow A[x := M]
$$

A term is in normal form if it contains no β-redexes.

The Krivine machine does not fully reduce terms. Instead, it evaluates to Weak Head Normal Form (WHNF), which is the form targeted by many functional languages.
This means it does not look inside lambda abstractions, so function bodies may still contain redexes. It also does not reduce arguments unless needed.

We only consider the head of a term. A WHNF is either:
- a lambda abstraction, or
- an application whose head is a variable

The syntax of WHNF can be described as:

$$
t ::= \lambda x.e \mid x e_{1} e_{2} \dots e_{n} 
$$

## Closure

The substitution used in β-reduction may duplicate terms and cause exponential growth during evaluation. To avoid this, the Krivine machine uses closures instead of direct substitution.
A closure is a term paired with its environment:

$$
c ::= \langle t, \rho \rangle
$$

An environment is a finite mapping from variables to closures:

$$
\rho ::= [] \mid \rho[x \mapsto c]
$$

I write $[]$ for the empty environment, and $\rho[x \mapsto c]$ for an environment extended with a binding from $x$ to closure $c$.

## Krivine machine

The state of the Krivine machine is a triple

$$
(M, \rho, S)
$$

where:
- $M$ is a term,
- $\rho$ is an environment,
- $S$ is a stack of closures.

The machine is defined by the following transition rules:

$$
(\lambda x. M, \rho, c \cdot S) \longrightarrow (M, \rho[x \mapsto c], S)
$$

$$
(M\ N, \rho, S) \longrightarrow (M, \rho, ( \langle N, \rho \rangle ) \cdot S)
$$

$$
(x, \rho, S) \longrightarrow (M, \rho', S)
\quad \text{where } \rho(x) = \langle M, \rho' \rangle
$$

The machine starts from an initial state:

$$
(M, [], \bullet)
$$

where $M$ is a closed term.

A computation terminates when no rule applies. In the standard presentation, this corresponds to reaching Weak Head Normal Form (WHNF).
