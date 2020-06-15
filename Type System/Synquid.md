# Synquid

Program Synthesis from Polymorphic Refinement Types

Nadia Polikarpova, Ivan Kuraj, Armando Solar-Lezama (MIT CSAIL, USA)

## Refinement Types

Types decorated with predicates from a decidable logic.

```
replicate :: n:Nat -> x:a -> {v:List a|len v = n}
```

Shape: underlying unrefined type

## Problem Definition

A synthesis problem is defined by

1. a goal refinement type $T$

2. a typing environment $\Gamma$

   contains type signatures of components available to the synthesizer, as well as any path conditions that can be assumed

3. a set of logical qualifiers $\mathbb{Q}$

   predicates from the refinement logic used as building blocks for unknown refinements and branch guards

$\beta$-normal $\eta$-long form: 

- $f: a \rightarrow b \rightarrow y$ 
- $\lambda a .\lambda b .f a b$

## Term Definitions

Horn Clause: Imply[Conjunction[Atomic-formula ...], Atomic-formula]