

# Lockpick Model

## 1. State Space and Operations

Let (N=64) and let
[
R=\mathbb Z_{16}.
]

A **state** is a triple
[
\mathbf s=(A,D,B)\in R^N\times R^N\times R^N,
]
where

* (A=(A_0,\dots,A_{N-1})^\mathsf T) is the first buffer,
* (D=(D_0,\dots,D_{N-1})^\mathsf T) is the dialer,
* (B=(B_0,\dots,B_{N-1})^\mathsf T) is the second buffer. 

The **initial state** is
[
A=\mathbf 0,\qquad D=\mathbf 0,\qquad B_i=i\bmod 16\quad(0\le i<N).
]


### 1.1 Turn operation

For any (u\in R^N), a turn adds (u) to the dialer:
[
(A,D,B)\longmapsto (A,D+u,B).
]

A unit turn at coordinate (i) is (u=e_i). By repetition, any dialer increment (u) can be realized. 

### 1.2 Left shift operation

Define (L) by
[
\begin{aligned}
A'*j&=
\begin{cases}
A*{j+1}, & 0\le j\le N-2,\
D_0, & j=N-1,
\end{cases}[4pt]
D'*j&=
\begin{cases}
D*{j+1}, & 0\le j\le N-2,\
B_0, & j=N-1,
\end{cases}[4pt]
B'*j&=
\begin{cases}
B*{j+1}, & 0\le j\le N-2,\
0, & j=N-1.
\end{cases}
\end{aligned}
]

So all three rows shift left; (D_0) enters (A_{N-1}); (B_0) enters (D_{N-1}); and the rightmost (B)-cell becomes (0). 

### 1.3 Combined move

For (u\in R^N), define
[
F_u(\mathbf s)=L(\mathbf s+\Gamma u),
\qquad
\Gamma u:=(0,u,0)\in R^{3N}.
]

Thus (F_u) means “turn by (u), then shift once.” We count each (F_u) as one **combined move**. Primitive counting instead treats the turn and the shift as separate moves. 

---

## 2. Output Controllability and Open-Loop Optimality

Let (T\in R^N) be a desired target output for the (A)-buffer.

### 2.1 Direct controllability construction

Every target (T) is reachable.

Start from the initial state. First apply one turn with
[
u=T,
]
so the dialer becomes (D=T). Then apply (N) left shifts and no further turns.

After the first shift, (T_0) enters (A_{N-1}). Each later shift moves previously inserted values one position left while the next dialer value enters from the right. After exactly (N) shifts, the full vector (T) has been flushed into (A), so
[
A=T.
]

Thus every target is reachable in:

* (N+1) primitive moves: one turn plus (N) shifts;
* (N) combined moves: one (F_T) followed by (N-1) copies of (F_0). 

### 2.2 Worst-case optimality in primitive moves

This (N+1) primitive bound is worst-case optimal.

Take the specific target
[
T=e_0,
]
so (T_0=1) and all other coordinates are (0).

Initially (A=\mathbf 0). A shift only inserts into (A_{N-1}), and then later shifts move that inserted value leftward one position per shift. Therefore for any (k<N), the coordinate (A_0) is still (0). Since the target has (T_0=1), at least (N) shifts are necessary.

Also, without at least one turn, the dialer remains zero forever, so (A) remains zero forever as well. Thus at least one turn is necessary.

Hence some targets require at least
[
N\text{ shifts}+1\text{ turn}=N+1
]
primitive moves. Since the construction above achieves (N+1), it is worst-case optimal. 

---

## 3. Guided Extension with Exact Shortest-Path Distance

Now suppose the target (T) is known explicitly, and let
[
\mathcal T={(A,D,B)\in R^{3N}:A=T}.
]

Define the exact controllability distance
[
\phi(\mathbf s)=\min\Bigl{m\ge 0:\exists u_1,\dots,u_m\in R^N
\text{ such that }
F_{u_m}\circ\cdots\circ F_{u_1}(\mathbf s)\in\mathcal T\Bigr}.
]

So (\phi(\mathbf s)) is the minimum number of combined moves needed to reach a goal state. 

### 3.1 Finiteness of (\phi)

For any state ((A,D,B)), choose first
[
u=T-D.
]
After that turn, the dialer becomes (T), and after the shift part of (F_u), the first coordinate of (T) enters (A_{N-1}). Then apply (N-1) more combined moves with turn (0). After (N) combined moves total, the buffer (A) equals (T).

Therefore
[
\phi(\mathbf s)\le N<\infty
]
for every state. Also,
[
\phi(\mathbf s)=0 \iff \mathbf s\in\mathcal T.
]
Since the full state space is finite, (\phi) can in principle be precomputed offline by breadth-first search. 

### 3.2 One-step optimality lemma

For any non-goal state (\mathbf s\notin\mathcal T),
[
\min_{u\in R^N}\phi(F_u(\mathbf s))=\phi(\mathbf s)-1.
]

This is immediate from optimality of (\phi): an optimal path must have some first move (u^*) reducing the remaining distance by (1), and no move can reduce it by more than (1), or else (\phi(\mathbf s)) would not be minimal. 

### 3.3 Guided algorithm with arbitrary tie-breaker

Let
[
d:R^N\times R^N\to\mathbb R_{\ge 0}
]
be any function satisfying
[
d(X,Y)=0 \iff X=Y.
]

At each state (\mathbf s_k), define the optimal first-move set
[
\mathcal U_{\mathrm{opt}}(\mathbf s_k)
======================================

{u\in R^N:\phi(F_u(\mathbf s_k))=\phi(\mathbf s_k)-1}.
]

Choose any (u_k\in\mathcal U_{\mathrm{opt}}(\mathbf s_k)) minimizing
[
d\bigl(\pi_A(F_u(\mathbf s_k)),T\bigr),
]
where (\pi_A) extracts the (A)-buffer.

Then apply (F_{u_k}). 

### 3.4 Termination proof

If (\phi_k=\phi(\mathbf s_k)), then by construction
[
\phi_{k+1}=\phi_k-1
]
at every non-goal step. Since (\phi_k) is a nonnegative integer, it reaches (0) in finitely many steps, and at that point (A=T).

So the algorithm terminates for **any** tie-breaker (d) satisfying only zero-equivalence. The role of (d) is merely to choose among already-optimal next moves; it does not carry the correctness proof. 

---

## 4. Why the Naive (k)-Based Enhancement Does Not Work

The proposed enhancement introduced a prefix progress variable
[
k(\mathbf s)=\max{k\in{0,\dots,N}:A_0,\dots,A_{k-1}\text{ match }T},
]
claimed that (k) is constructively computable from the distance oracle, and then used (k) to define a safe-action filter and a failure-free fallback. 

That construction is not valid as stated.

### 4.1 The shift does not preserve a fixed prefix

Under the true shift rule,
[
A'*j=A*{j+1}\quad(0\le j\le N-2),\qquad A'_{N-1}=D_0.
]

So a shift moves the entire (A)-buffer left and discards (A_0). Therefore a matched prefix in the left part of (A) is not preserved under further shifts. The claimed lemma that “previously fixed prefix remains unchanged due to shift structure” is false for this dynamics. The proposed prefix-growth proof therefore collapses. Compare the actual shift rule with the enhancement’s lemma.  

### 4.2 The oracle assumptions are too weak to compute (k)

The enhancement assumes an unknown target (T) accessible only through an oracle value (d(A,T)) with the single condition
[
d(X,Y)=0 \iff X=Y.
]

But this alone does not let one determine which coordinates of (A) match (T), nor recover (T_k) coordinate by coordinate. So neither the progress variable (k(\mathbf s)) nor the stage-wise “probe (T_k)” step is justified from the stated assumptions.  

### 4.3 The safe-action filter is not executable

The enhancement defined
[
\mathcal U_{\mathrm{safe}}(\mathbf s)={u:k(F_u(\mathbf s))\ge k(\mathbf s)}.
]

But if (k) itself is not observable from the oracle assumptions, then this filter is not constructively available either. So the “every executed move is safe” guarantee was assumed rather than proved. 

### 4.4 The (O(N^2)) fallback is not the correct strongest guarantee

In the known-target regime, the original model already gives a direct worst-case optimal open-loop method with (N+1) primitive moves. So replacing that by an (O(N^2)) fallback is not an improvement in rigor or guarantee. 

---

## 5. Correct Failure-Free Hybrid Formulation

We now state the corrected hybrid model.

### 5.1 Setting

The target (T\in R^N) may be unknown in explicit coordinate form. We only assume access to a goal-test-type oracle
[
d(A,T),
\qquad d(X,Y)=0 \iff X=Y.
]

ML components may propose promising moves or rank states, but they are not trusted for correctness.  

### 5.2 State graph

Consider the directed graph whose vertices are all states in
[
R^{3N},
]
and whose outgoing edges from (\mathbf s) are
[
\mathbf s\to F_u(\mathbf s)\qquad(u\in R^N).
]

This graph is finite because the state space is finite. Also, from the controllability argument above, for every target (T), some goal state with (A=T) is reachable from the initial state. 

### 5.3 Deterministic complete fallback

A fully rigorous failure-free fallback is:

1. Maintain a set of visited states.
2. Start from the initial state.
3. At every state, test whether (d(A,T)=0). If yes, stop.
4. Otherwise, expand outgoing moves (F_u) according to some enumeration of (u\in R^N).
5. Never revisit an already explored state.
6. Continue by breadth-first search, depth-first search with cycle elimination, or any complete systematic graph search.

Because the graph is finite and a goal state is reachable, this fallback must eventually terminate successfully. This is the correct constructive guarantee under the stated oracle assumptions. It does not require coordinate recovery of (T), and it does not assume an invalid monotone prefix quantity. This is the repaired version of the “failure-free” claim. It uses the original reachability result as the existence guarantee and the finiteness of the state graph as the completeness guarantee. 

---

## 6. ML-Augmented Controller with Guaranteed Correctness

Now let ML enter only as an acceleration layer.

### 6.1 Learned components

Possible learned components include:

* a policy (\pi_\theta(\mathbf s)) proposing promising actions (u),
* a value heuristic (\hat\phi(\mathbf s)),
* a ranking function on frontier states,
* or a learned pruning preference that is advisory only.

The ML system is allowed to influence **search order**, but not logical correctness. This preserves the intended “ML helps, but does not need to be trusted” principle from the enhancement. 

### 6.2 Hybrid action rule

At any point:

* use ML to rank which frontier node or which outgoing actions to try first;
* still maintain the deterministic visited-state logic;
* if ML suggestions fail, continue the systematic complete exploration.

So ML changes only efficiency, never reachability or termination. 

---

## 7. Main Theorem (Corrected Failure-Free Guarantee)

**Theorem.**
Assume the lockpick dynamics defined above, a target (T\in R^N), and an oracle (d) satisfying
[
d(X,Y)=0 \iff X=Y.
]
Assume further that the controller uses any complete deterministic graph-search fallback over the state graph (\mathbf s\to F_u(\mathbf s)), with cycle elimination by visited states, while allowing ML to rank actions or states arbitrarily. Then the hybrid controller terminates in finite time at some state with (A=T), independently of whether the ML component is correct.

### Proof

The state space (R^{3N}) is finite. By controllability, from the initial state there exists at least one finite path to a state satisfying (A=T). The complete fallback systematically explores reachable states without revisiting already explored ones. Therefore it eventually reaches a goal state. The oracle condition (d(A,T)=0\iff A=T) ensures that the controller can recognize success exactly when it occurs. Since ML only affects the order in which states or actions are considered, not the completeness of exploration, ML cannot destroy termination or correctness. Hence the controller always halts successfully in finite time. ∎

This is the rigorous replacement for the invalid (k)-monotonicity theorem.  

---

## 8. Complexity and Interpretation

### 8.1 Exact (\phi)-based control

If (\phi) were available exactly, then one can follow an optimal path in the combined-move metric and terminate in exactly (\phi(\mathbf s_0)\le N) combined moves. This is mathematically clean but not practical at full scale, since exact precomputation is infeasible on the full state space. 

### 8.2 Open-loop construction

If the target is known explicitly, the simple open-loop strategy reaches the target in (N+1) primitive moves, worst-case optimally. 

### 8.3 Correct hybrid guarantee

If the target is only available through a weak oracle and ML is used for speed, then the strongest fully rigorous general guarantee is:

* **correctness** from controllability plus complete finite-state search,
* **exact optimality** from (\phi) when available,
* **practical acceleration** from ML ordering heuristics.

That is the proper three-layer interpretation of the system. It preserves the spirit of the enhancement while keeping every guarantee honest.  

---

## 9. Final Unified Interpretation

The corrected model has three distinct layers:

### (A) Exact optimal control

[
\phi(\mathbf s)
]
gives the true shortest-path distance in the combined-move graph. It yields a clean universal termination proof when available exactly. 

### (B) Deterministic failure-free control

A complete search over the finite state graph, with visited-state elimination and oracle-based goal testing, gives a genuinely constructive guarantee that does not depend on ML correctness. This replaces the invalid prefix-based fallback.  

### (C) Learned acceleration

ML may rank nodes, propose actions, or prioritize expansions. It can improve expected runtime, but it is not part of the logical proof of success. 


