# Week 9

**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

**Dates:** 08-30 to 09-05

## Goals
- Draft up a summary of my project in LaTeX format
- Consolidate numerical results and insights learned


## Approach and Implementation
This week I drafted the project write-up in LaTeX. It follows a NeurIPS-style preprint layout with sections for the problem, related work, approach, results and discussion. Writing it forced me to formalize ideas that had only lived in my notes:

- *The fidelity gap.* I defined the mismatch between $`F(\mathcal{A})`$ and $`\mathcal{B}`$ as a symmetric Chamfer distance. Its two terms separate the image *drifting off* $`\mathcal{B}`$ from the image *failing to cover* $`\mathcal{B}`$, the latter being the surjectivity issue.
- *Why pullback geodesics go straight.* The scaled isometry of Wurgaft et al. only constrains $`G_F`$ on the tangent bundle of $`\mathcal{A}`$. Since $`\mathrm{rank}\, J_F \le \dim \mathcal{Y}`$, $`J_F`$ has at least a 56-dimensional null space in PCA(64), so moving off $`\mathcal{A}`$ is nearly free and unconstrained geodesics cut across it. This is exactly what the confinement metric removes.
- *A fidelity certificate.* By the triangle inequality, at every point of the path

$$\min_{b \in \mathcal{B}} \lVert F(\hat{x}(t)) - b \rVert_2 \;\le\; L\, d_{\mathcal{A}}(t) + d_{\mathcal{B}}(t),
$$

so the optimized distance terms bound the fidelity gap along the traversed arc. The tangential term is what guarantees the projections sweep the whole arc rather than skipping parts of it.

- *Hellinger ↔ Fisher.* Because $`F`$ outputs $`\sqrt{p}`$, the pullback metric is a quarter of the Fisher information of the output distribution: $`G_F = \tfrac{1}{4} I_{\text{out}}`$. Pullback length therefore measures how fast the model's prediction becomes distinguishable as you steer.

I also cleaned up the derivations themselves: the implicit-function backward pass and the PCA(64) working subspace.


## Results
- A complete draft with abstract, method and bibliography. The figures are still placeholders and the lifted-path baseline number is still missing.
- Consolidated four tables:
  - layer localization by activation patching,
  - pullback vs. linear baselines over all 21 weekday pairs,
  - the confinement ablation,
  - the full $`(\lambda_{a}, \lambda_{b})`$ fidelity sweep.
- The main takeaway is now stated precisely. In the weekday modality, composite-loss geodesics stay within about $`10^{-5}`$ MSE of the activation spline and $`5 \times 10^{-3}`$ of the behavior spline. Where the two conflict, the trade-off localizes where $`F`$ deforms the geometry.


## Notes


