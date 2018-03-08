---
title: theory of computation
tags: computation
---

- automata, computability, complexity: What are the fundamental capabilities and limitations of computers?
- complexity: What makes some problems computationally hard and others easy?
  - no answer, but some research make
  - cryptograph
- computability: solvable or not
- automata: mathematical models of computation

## regular languages

- finite state machine or finite automaton
- probabilistic counterpart: Markov chains
- formal definition of finite automaton: 5 tuple $(Q, \Sigma, \delta, q_0, F)$
- set A is language of machine M iif M accepts every element of A, write L(M) = A
- a language is called a `reguler language` if some finite automaton recognizes it. L(M)
- design: `reader as automaton`
- regular operations: `union`, `concatenation`, `star.` The collection of regular languages is closed under all three of the regular operations
- regular languages is close under union:
  - $A = A1 \cup A2$ is also regulr language
  - let's construct an finite machine $M = (Q, \Sigma, \delta, q_0, F)$ use $M = (Q_1, \Sigma_1, \delta_1, q_1, F_1)$ and $M2 = (Q_2, \Sigma_2, \delta_2, q_2, F_2)$ (recognizes A1 and A2).
  - $Q = {(r_1, r_2) | r_1 \in Q_1\ and\ r_2 in Q_2}$
  - $\Sigma = \Sigma_1 | \Sigma_2$
  - $\delta((r_1, r_2), a) = (\delta_1(r_1, a), \delta_2(r_2, a))$
  - $q_0 = (q_1, q_2)$
  - $F = {(r_1, r_2) | r_1 \in F_1\ or\ r_2 \in F_2}$
  - NFA
- regular languages is close under concat
  - $A = A1 \circ A2$ is also regulr language
  - an \epsilon between M1 and M2 
- deterministic (DFA), nondeterministic (NFA, parallel computation, \delta is multiple to multiple), \epsilon (empty transition)
  - NFAs and DFAs are same powerful
  - equivalent DFA of NFA
- closure under the regular operations by nondeterministic
  - union: $delta-$ transition to N1 and N2
  - concat: all F(N1) $\delta-$ transition to N2
  - star: all F(N1) $\delta-$ transition to $q_0$
- regular expressions
  - is equivalence with finite automata
- nonregular languages
  - limitations: $B = {0^n1^n | n \geq 0}$
  - pumping lemma: nonregularity

## context-free languages

- formal definition of a context-free grammar $(V, \Sigma, R, S)$
- pushdown automata (PDA)
  - NFA + stack is equivalent to context-free grammar
  - not DFA + stack

## The Church-Turing Thesis

- turing machine: infinite tape, read/write head
- formal definition: $\delta \Rightarrow Q \times \Gamma \longrightarrow Q \times \Gamma \times {L, R}$
  - machine is in state `q` and head is `a`
  - if $\delta(q,a) = (r, b, L)$
  - machine write `b` replace `a`, and go to state `r`
  - `L or R` indicates the head is direction after write
- language is Turing-decidable (or decidable) if some Turing machine decides it. never loop
- language is Turing-recognizable if some Turing machine recognizes it. may loop
- variants:
  - multitape turing machine
  - NTM
  - enumerator

## definition of algorithm

- thne 10th hilbert's problem: whether a polynomial has an integral root. There is no such algorithm.
- $D = {p | p \ is \ a \ polynomial \ with \ an \ integral \ root}$, whether the set `D` is decidable.
- A Turing Machine is a precise model for the definition of algorithm.

## Time Complexity

## Class P
- polynomial and exponential time: deterministic, non-deterministic

## Class NP
- NP is the class of languages that are solvable in polynomial
time on a nondeterministic Turing machine
- np-completeness