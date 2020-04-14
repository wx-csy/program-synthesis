# Sketch

**Program Sketching**

STTT'13

Keywords: SAT/SMT applications, Constraint-based synthesis, Sketching, Synthesis

Frontend: https://bitbucket.org/gatoatigrado/sketch-frontend/src/default/

Backend: https://bitbucket.org/gatoatigrado/sketch-backend/commits/

实质：利用符号执行合成程序

## Basic Notation

- $L$: the set of variables in the program. $x$ is used to refer to an arbitrary variable.

- $H$: the set of holes in the program. All holes are assumed to be uniquely labeled.
- $T$: the set of calling contexts.
- $\mathbf{\Phi} = H \times T \rightarrow \mathbb{Z}$: the set of control functions.
- $P(\mathbf{\Phi})$: the powerset of $\mathbf{\Phi}$.
- $\Psi = \mathbf{\Phi} \rightarrow \mathbb{Z}$: the set of parameterized values. A parameterized value produces an integer for each control.
- $\Sigma = L \rightarrow \Psi$:  the set of all states. A state $\sigma$ maps each variable to a parameterized value.

![image-20200414145612275](image-20200414145612275.png)

## Denotational Semantics

![image-20200414145818650](image-20200414145818650.png)

![image-20200414145835479](image-20200414145835479.png)

### Comparison with Traditional Denotational Semantics

|            | Traditional Semantics                       | Synthesis Semantics                                          |
| ---------- | ------------------------------------------- | ------------------------------------------------------------ |
| Expression | $A[expr]: \Sigma \rightarrow \mathbf{\Psi}$ | $A'[expr]: \Sigma \rightarrow \mathbb{Z}$                    |
| Command    | $C[cmd]: \Sigma \rightarrow \Sigma_{\bot} $ | $C'[cmd]: (\Sigma, P(\mathbf{\Psi})) \rightarrow (\Sigma, P(\mathbf{\Psi})) $ |

## Synthesis Algorithm

![image-20200414155400401](image-20200414155400401.png)

