# EUSolver

**Scailing Enumerative Program Synthesis via Divide and Conquer**

TACAS'17, part of ETAPS'17

Source code: https://bitbucket.org/abhishekudupa/eusolver/

## General Comments

1. Counterexample-guided synthesis => PbE

2. Generate terms and predicates separately

3. Combine terms and predicates with decision tree

4. Verify solution with SMT Solver

5. From reviewers of TACAS'17:

   > One concern that might arise while reading this paper is that the term "divide and conquer" is used to describe the presented DCSolve algorithm. As opposed to the usual definition, the DCSolve algorithmdoes not feature a recursive subdivision of the original (synthesis) problem. While the problem is being divided into two different domains of terms and predicates, the algorithm still iteratively expands sets of terms and predicates until a matching solution is found. The recursive nature of the algorithm is missing here. The use of the term "divide and conquer" that is already present in the title of this paper might therefore not be the best fit and confuse readers initially.

## The Syntax-Guided Synthesis (SyGuS) Problem

SyGuS: **syntax restriction** (as CFG grammar) $G$, **semantic restriction** (as SMT formula) $\Phi$

Example: the maximum problem

- Syntax:

  ```
  S ::= T | if (C) then T else T
  T ::= 0 | 1 | x | y | T + T
  C ::= T <= T | C and C | not C
  ```

- Semantics:

  $$ \forall x, y: f(x, y) \geq x \wedge f(x, y) \geq y \wedge (f(x, y) = x \vee f(x, y) = y) $$

- Solution:

  `if x <= y then y else x`

## The Enumerative Solver

Input: grammar $G$, specification $\Phi$

Output: expression $e$, s.t. $e \in [\![G]\!] \wedge e \vDash \Phi$

```
pts = {}
while true :
	for e in Enumerate(Grammar, pts) :
        if e doesn't satisfy Spec on pts : continue
        cexpt = Verify(e, Spec)
        if cexpt = null : return e
        pts += cexcept
```

## The Divide-and-Conquer Enumeration Algorithms

1. Splitting grammar into **conditional expression grammar** $G = \langle G_T, G_P \rangle$:
   - term grammar $G_T$: generates expressions of target type
   - predicate grammar $G_p$: generates Boolean expressions
2. Enumerating terms and predicates separately
3. Combining terms and predicates with **decision tree**

### The Algorithm

Input: conditional expression grammar $G = \langle G_T, G_P \rangle$, specification $\Phi$

Output: expression $e$, s.t. $e \in [\![G]\!] \wedge e \vDash \Phi$

```
pts = {}
while true :
	terms = {}; preds = {};
	while the union of cover[t] over all terms t != pts :	// term solver
		terms += NextDistinctTerm(pts, terms)
	while DT = null :										// unifier
		terms += NextDistinctTerm(pts)
		preds += Enumerate(Gp, pts)
		DT = LearnDT(terms, preds)
	e = expr(DT); cexpt = verify(e, spec);					// verifier
	if cexpt = null : return e
	pts += cexcept
```

### Decision Tree Learning

Input: `pts, terms, cover, preds`

Output: decision tree `DT`

```
if exists term t such that pts is a subset of cover[t] :
	return LeafNode[t]
if preds = {} : return null
p = pick predicate from preds
L = LearnDT({pt|p[pt]}, terms, cover, preds \ {p})
R = LearnDT({pt|!p[pt]}, terms, cover, preds \ {p})
return InternalNode[p, L, R]
```

## Extensions and Optimizations

### The Anytime Extension

- Do not stop upon finding a valid answer; let synthesizer generates more solutions.
- Larger predicates and terms may result in simpler answers.

### Branch-wise verification

- Verifying each branch separately might be faster than verifying the entire expression.
- The cost of SMT solver grows exponentially to the size of the expression.

