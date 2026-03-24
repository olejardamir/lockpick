 

# Revised Distance-Integrated Version

## 7. External Distance Function Based on Hex Comparison

Let (F) be the external deterministic map from a 64-hex input key (K) to a 48-hex output string:
[
F: \mathcal K \to {0,\dots,15}^{M},
\qquad M=48.
]

Let the fixed target output be
[
T=(T_0,\dots,T_{M-1})\in {0,\dots,15}^{M}.
]

For any candidate key (K), define
[
X(K)=F(K)=(X_0(K),\dots,X_{M-1}(K)).
]

We now define the external target-distance (D_{\mathrm{ext}}(K)) by comparing (X(K)) to (T).

---

## 7.1 Pairwise position-value cost

For positions (i,j\in{0,\dots,M-1}), define the pairwise comparison cost
[
c_K(i,j)
========

\alpha,|X_i(K)-T_j|
+
\beta,|i-j|,
]
where

* (\alpha>0) weights hex-value difference,
* (\beta>0) weights positional displacement.

Thus a candidate output digit is considered close to a target digit when

* their hex values are numerically close, and
* their positions are close.

---

## 7.2 Optimal one-to-one alignment

Let (S_M) denote the set of permutations of ({0,\dots,M-1}).
For each candidate key (K), define the minimum-cost assignment
[
\pi_K^\ast
\in
\arg\min_{\pi\in S_M}
\sum_{i=0}^{M-1}
\Bigl(
\alpha |X_i(K)-T_{\pi(i)}|
+
\beta |i-\pi(i)|
\Bigr).
]

This gives the best one-to-one matching between candidate-output digits and target digits.

Define the matched per-coordinate costs
[
m_i(K)
======

\alpha |X_i(K)-T_{\pi_K^\ast(i)}|
+
\beta |i-\pi_K^\ast(i)|.
]

---

## 7.3 Sorted weighted assignment distance

Sort the matched costs in nondecreasing order:
[
m_{(1)}(K)\le m_{(2)}(K)\le \cdots \le m_{(M)}(K).
]

Choose increasing positive weights
[
w_1\le w_2\le \cdots \le w_M,
]
for example
[
w_r=r
\qquad\text{or}\qquad
w_r=r+1.
]

Define the external distance by
[
D_{\mathrm{ext}}(K)
===================

\sum_{r=1}^{M} w_r, m_{(r)}(K).
]

Interpretation:

* (D_{\mathrm{ext}}(K)\ge 0),
* smaller means closer to target,
* (D_{\mathrm{ext}}(K)=0) iff (X(K)=T).

The sorting step makes larger mismatches contribute more strongly after weighting, which preserves the spirit of your earlier metric while extending it to cross-position comparisons.

---

## 7.4 Exact-match property

Because all costs are nonnegative,
[
D_{\mathrm{ext}}(K)=0
]
holds if and only if every matched cost is zero, which requires
[
X_i(K)=T_{\pi_K^\ast(i)}
\quad\text{and}\quad
i=\pi_K^\ast(i)
]
for all (i). Hence
[
D_{\mathrm{ext}}(K)=0
\iff
X(K)=T.
]

So this metric is compatible with exact target verification.

---

## 8. External Directional-Search Hypothesis with Implemented Distance

The earlier abstract directional layer now becomes concrete.

For ordered search variable (K), define the true external distance by the metric above:
[
D_{\mathrm{ext}}(K)
===================

\sum_{r=1}^{M} w_r, m_{(r)}(K),
]
where the (m_{(r)}(K)) arise from the minimum-cost assignment between (F(K)) and the target (T).

We then test whether moving in an ordered search direction improves this exact metric.

For any horizon (h\ge 1),
[
Y_h(K,d)
========

\mathbf 1{D_{\mathrm{ext}}(K+dh)<D_{\mathrm{ext}}(K)},
\qquad d\in{+1,-1}.
]

Thus
[
Y_h(K,d)=1
]
exactly when moving (h) steps in direction (d) gives a candidate whose output is closer to the target under the new distance.

This makes the directional labels fully consistent with the actual metric.

---

## 9. Multi-horizon labels with the implemented metric

For horizons (h_1,\dots,h_m), define
[
Y(K,d)
======

\bigl(
Y_{h_1}(K,d),\dots,Y_{h_m}(K,d)
\bigr),
]
where each label is computed from
[
D_{\mathrm{ext}}(K)
===================

\sum_{r=1}^{M} w_r, m_{(r)}(K).
]

The learning problem is now:

estimate whether moving left or right in key space is likely to decrease the **sorted weighted assignment distance** between the candidate output and the target output.

So the model is no longer trained against an undefined “distance”; it is trained against your explicit metric.

---

## 10. Directional contrast with the implemented metric

The uncertainty logic remains unchanged, but now all improvement probabilities are defined using your actual distance:
[
\Delta_h(K)=\hat p_h(K,+1)-\hat p_h(K,-1),
]
where
[
p_h(K,d)=P\bigl(D_{\mathrm{ext}}(K+dh)<D_{\mathrm{ext}}(K)\bigr).
]

So every directional decision is grounded in whether the move reduces the assignment-based hex-distance to target.

---

## 11. Full integrated definition

The complete external metric is therefore:

[
X(K)=F(K),
]
[
c_K(i,j)=\alpha |X_i(K)-T_j|+\beta |i-j|,
]
[
\pi_K^\ast \in \arg\min_{\pi\in S_M}\sum_{i=0}^{M-1} c_K(i,\pi(i)),
]
[
m_i(K)=c_K(i,\pi_K^\ast(i)),
]
[
m_{(1)}(K)\le \cdots \le m_{(M)}(K),
]
[
D_{\mathrm{ext}}(K)=\sum_{r=1}^{M} w_r,m_{(r)}(K).
]

Then directional improvement is defined by
[
D_{\mathrm{ext}}(K+dh)<D_{\mathrm{ext}}(K).
]

That is the proper mathematical insertion.

---

## 12. What this fixes

This implementation gives the external layer a real metric instead of an unspecified target-distance. It also makes the later directional-search formulas internally consistent, because all labels, probabilities, contrasts, and corridor scores now refer to the same precise distance. The uploaded model explicitly separated the internal lockpick theorem from the external directional layer, and this insertion keeps that separation intact while making the external layer concrete. 

---

## 13. Important note

This metric is appropriate only for the **external search/output-comparison layer**. It should **not** replace the exact internal controllability distance
[
\phi(\mathbf s),
]
because (\phi) is a shortest-path distance in the internal combined-move graph, while your new metric is a comparison distance between a produced hex output and a target hex output. Those are different objects.

So the correct structure is:

* keep (\phi(\mathbf s)) for the exact internal theorem,
* replace abstract (D_{\mathrm{ext}}(K)) with your new assignment-based distance in the external heuristic layer.

That is the mathematically proper implementation.
