# Euphony

**Accelerating Search-Based Program Synthesis using Learned Probabilistic Models**

PLDI'18

Keywords: Synthesis, Domain-specific languages, **Statistical methods**, **Transfer learning**

## Problem to Solve

Syntax-Guided Synthesis (SyGuS)

- Syntactic specification: CFG
- Semantic specification: I/O Example (PbE paradigm)

## Existing Method: Enumeration with an Unguided Search

Generator-verifier framework

Unguided enumeration **by size** → enumerate too many **unlikely** programs → fail in *large-scale* search space

Insight: guide the search towards likely programs!

## Contributions

- Accelerating the program synthesis with probabilistic model (via A* search on graph of sentential forms)
- Learning a probabilistic model which *generalizes* well (via **transfer learning**)

## Definition

### Probabilistic CFG (PCFG)

$Pr(S \rightarrow \alpha | n) = x $, where $n$ is a sentential form

### Probabilistic Higher-Order Grammar (PHOG)

$S[context] \rightarrow \alpha, Pr = x $, where $context$ is the context information

## Algorithms

### CEGIS with Guided Search

```
CEGIS(G, Constraint)
	pts := {}
	repeat
		P := weighted_search(G, pts, Constraint)
		cex := verify(P, Constraint)
		if cex = {} then
			return P
		end if
		pts := pts + {cex}
	until false
```

### Weighted Enumerative Search (A* search)

```
Weighted_Search(G. pts, Constraint, g)
	Q := {(S, 0, g(S))}
	while Q is not empty do
		remove (n, cf, cg) whose cf+cg is minimal from Q
		if n is a sentence and satisfies Constraint then
			return n
		end if
		for all n' s.t. n -> n' (leftmost one-step derivation) do
			insert (n', cf+w(n->n'), g(n')) into Q
		end for
	end while
```

#### Heuristic Function

$$ g(n) = - \sum_{n_i \in N} \log_2 h(n_i) $$, where $N$ is the set of non-terminals.

$h(A)$ is the maximum probability for deriving non-terminal $A$ to a sentence:

$$ h(A) = \max_{A \rightarrow \beta, c \in C} \left(q(A \rightarrow \beta | c) \times \prod_{\beta_i \in N} h(\beta_i) \right)$$

### Pruning with Equivalence Classes

Two sentential forms $n_i, n_j$ are weakly equivalent modulo $pts$ iff $n_i = n_j$ or

$$ \exists P_i, P_j \in \mathbb{P}, P_i < n_i, P_j < n_j, \forall x \in pts, [[p_i]](x) = [[p_j]](x), \\ n_i[P_i/\varepsilon] = n_j[P_j/\varepsilon]$$

where $<$ denotes the substructure relation.

All weakly equivalent sentential forms are merged, only preserving the one with maximum probability.

> My note: the probability of weakly equiv. sentential forms can be summed instead of preserving the maximum one.
>
> Rationale: weakly equiv. sentential forms have isomorphic subtrees (modulo the equivalent substructure). The target is to find program with maximum probability. Hence the prob. equivalent sentential forms can be summed.

## Pivot Grammar Example

```
S -> x | S + S
   | Rep(S, S, S)
   | const_IO | const_I
   | const_O  | const_NIL
```

- `const_IO` represents the set of substrings of all strings in $I \cap O$
- `const_I` represents the set of substrings of all strings in $I$
- `const_O` represents the set of substrings of all strings in $O$
- `const_NIL` represents all remaining strings