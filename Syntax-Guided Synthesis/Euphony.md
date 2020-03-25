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

## Q & A

Q: 启发式函数的设计？

A: A*算法要求启发式函数是真实代价的乐观估计（即估计代价不能超过真实代价）才能得到正确解。

启发式函数的设计是基于这样一个简单的想法：一个正确的解必然是满足语法限制的。因此，正解的最小负对数概率不会小于所有语法正确的解的负对数概率的最小值；从而可以使用后者作为前者的一个乐观估计。

对每个非终结符$A \in N$，定义$h(A)$为

$$
h(A) = \max_{A\rightarrow \beta \in R, c \in C} q(A \rightarrow \beta | c) \times \prod_{\beta_i \in N} h(\beta_i)
$$

上述定义的直观含义是，将$A$推导至终结符号串的最大概率。上面的定义实际上是一个方程，可以用迭代法求出每个$h(A)$的值。

然后，可以定义每个句型的估价函数：
$$
g(n) = -\log_2\prod_{n_i \in N} h(n_i) = -\sum_{n_i \in N} \log_2 h(n_i)
$$
即将$n$中的每个非终结符推导到终结符号串的概率的最大值的负对数。$g(n)$的意义是将句型$n$推导得到一个满足语法限制的程序的最大概率（但这个程序不一定满足语义限制），因此$g(n)$是当前代价的乐观估计，即$g(n) \leq g^*(n)$ (Theorem 3.3)。

Q: 为什么只允许最左推导？

A: 限制最左推导，可以保证得到一个句子的推导路径是唯一的，即句型图是一棵有根树。这样可以直接使用链式法则将各步的条件概率相乘得到总的概率。此外，使用最左推导可以保证在展开语法树时，当前非终结符节点的左兄弟节点已被展开，这样可以将左兄弟的内容作为上下文信息的一部分。