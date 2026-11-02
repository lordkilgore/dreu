# Week 7

**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

**Dates:** 08-16 to 08-22

## Goals
- Run initial confinement tests
- Evaluate composite fidelity losses


## Approach and Implementation
This week I implemented the confinement metric,

$$G_C^{(\mathcal{M})}(x) = (\lambda_N - \lambda_T)\,P_N(x) + \lambda_T I,$$

which penalizes motion normal to the manifold. It keeps a small tangential term so that on-manifold paths still have a well-defined length.

Finding the nearest on-manifold point was the main obstacle. Newton’s method stalled in local minima, so I split the spline parameter interval into one search window per waypoint, and used an implicit-function backward pass to recover the gradient through the search. I then ablated how the on-manifold point is selected, where the metric is evaluated, and whether the windows are fixed or move with the waypoints.

With the metric validated, I combined both manifolds into the composite loss $`\lambda_{a} \mathcal{L}_{\mathcal{A}} + \lambda_{b} \mathcal{L}_{\mathcal{B}}`$ and swept $`(\lambda_{a}, \lambda_{b})`$ across the weekday, month and age modalities. To make the sweep practical, I batched all waypoints into a single forward/backward pass.


## Results
- Partition-midpoint selection with fixed windows worked best, and a small tangential weight ($\lambda_T = 0.05$) stopped waypoints from jumping ahead. The resulting geodesics track both the activation and behavior splines closely.
- In the weekday modality, a single composite-loss path stays close to both splines: about $10^{-5}$ MSE to $\mathcal{A}$ and $5\times10^{-3}$ to $\mathcal{B}$. Raising $\lambda_b$ trades activation fidelity for behavior fidelity, so the two terms are visibly in tension.


## Notes


