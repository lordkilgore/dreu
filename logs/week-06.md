**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 6

**Dates:** 8-9 to 8-15

## Goals
- Formulate Riemannian Penalty Confinement Metric
- Implement Confinement Metric
- Pivot Project Narrative

## Approach and Implementation
Revisiting the pullback metric, I discovered that the induced geodesics weren't as promising as the paper had led me to believe. In the most generous sense, the metric induces geodesics whose image 'hugs' the boundary of the image of the map around the straight line between two points; e.g. the geodesics map to the projection of the straight-line path onto the image of the map.

$$L_{G_F}\[ \gamma \] = L_Y[F \circ \gamma] = L_2 [F \circ \gamma]$$

Given this, I knew the project had to pivot since there was nothing truly special obtained through the pullback. Instead, I decided to recast the pullback as a form of regularization: given a geometric prior on the activation space, we use the pullback to anchor the image of this prior. In addition to this, I revisited the literature and started thinking about how these priors (in both the activation and behavior spaces) could be mutually respected by steering paths.

The result was this: steering along an activation manifold induces drift from the behavior manifold and steering along the behavior manifold induces drift vice versa. The solution is to then optimize for a path which drifts least wrt both geometric priors. Measuring "drift" first consisted of measuring potential distances of the form $||\gamma(t) - \hat{x}(\gamma(t))||_2$, however this need not respect the curvature of the manifold and instill a 0-1 characteristic to the loss. We amend this by introducing a "local-tangent" term $G_C ^{(\mathcal{M})}$, which uses the first-order tangent space information from the manifold to estimate curvature (see Notes for a detailed description).

$$\arg \min _{\gamma}  \, \int_0 ^1 \lambda_a \mathcal{L}_\mathcal{A}(\gamma(t)) +\lambda_b \mathcal{L}_\mathcal{B} (\gamma(t)) \, dt$$

Not only does this framework contribute to better integration of geometric priors in steering tasks, but it also provides a lens for interpretability. The loss on an optimized path decomposes into activation and behavior manifold error signals across the path parameter. This lets practitioners study where in activation space is the model morphologically invariant wrt the activation manifold and where it pushes the path towards the behavior manifold. Where $F$ happens to be locally faithful, e.g. where moving along the activation manifold is compatible with the behavior manifold (like the scaled isometry regime), the optimizer can drive both signals toward $0$. Where $F$ deforms the geometry or breaks the correspondence between activation and behavioral structure, $\gamma$ must sacrifice fidelity to one manifold to preserved fidelity for the other.

We can resolve this behavior at a higher fidelity by cascading these manifolds across LM layers. This, in effect, provides a geometric lens for layer localization methods in interpretability.

$$\arg \min_{\gamma} \int _0 ^1 [\sum _{\ell = \ell_0} ^{L-1}\lambda_\ell \mathcal{L}_{\mathcal{A}_\ell}(\gamma_{\ell}(t))] + \lambda_{b} \mathcal{L}_{\mathcal{B}}(\gamma_L(t)) \, dt$$

I have begun experimenting with the confinement metric, but I figure it would be more cohesive if I report the results in the next week's log. The rest of the project is delineated below:

  1. **Confinement Metric.** Perform ablations and refinements along the following design axes: (1) on-manifold point selection, (2) metric evaluation points, (3) distance potential term, (4) tangential-normal weighting.
	2. **Composite Fidelity Loss.** Form the composite loss and evaluate the fidelity across all modalities (weekday, age, month, shape) as well as the steering strength (using prior metrics + seeing if model inert to directions orthogonal to manifold). Formalize results with statistical testing (review literature on where applicable) and over an expectation of carriers.
	3. **Layer Localization.** Evaluate per-layer profiles in the arithmetic and summarization modalities across a cascade and compare results with activation patching (and perhaps a layer probe **IF TIME PERMITS! LOW PRIORITY**). Formalize results with statistical testing (review literature on where applicable) and over an expectation of carriers.
	4. **Behavior-Unsupervised Metric**. Perform ablations and refinements along the following design axes: (1) pullback, (2) pullback regularization weighting, (3) tangential-normal weighting, (4) distance potential term.
	5. **Behavior-Unsupervised Loss**. Form the composite loss and evaluate the smoothness and redundancy of induced trajectories (formalized statistically later) across weekday, age, month, and shape modalities.

## Results
- Project narrative respun

## Notes

### Local-Tangent Manifold Confinement
Consider a point $x \in \mathcal{M}_\ell$ and its tangent space $T_x \mathcal{M}_\ell < \mathbb{R}^{d_{\text{model}}}$, i.e. the subspace 'tangent' to the manifold up to translation. We can use this tangent space as a local, linear approximation of the manifold. 

Let $P_T (x)$ be the projector onto $T_x \mathcal{M}_\ell$, where $\text{rank}(P_T)$ must necessarily be $d << d_{\text{model}}$, with eigenvalues $1$ for directions on the tangent space and $0$ elsewhere. To first order, however, this makes off-manifold directions cheap, so we look at the projector onto the orthogonal complement, $P_N(x) = I_{d_\text{model}} - P_T(x)$, which thus has rank $d_{\text{model}} - d$.

However, using $P_N$ alone makes on-manifold paths indistinguishable -- the notion of a **shortest path** along $\mathcal{M}_\ell$ is no longer clear.  Thus, we must also include a term measuring these tangential lengths via $P_T$. Hence, the ideal, composite loss becomes the following. 

$$G_C^{(\mathcal{M})} = \lambda _N P_N+ \lambda_T P_T = (\lambda_N -\lambda_T)P_N +\lambda_TI$$

Interpreting $\lambda_T >0$ as a regularization weight, we now introduce a distance potential as a way to anchor waypoints near the manifold, where these tangent approximations are non-pathological.

$$\mathcal{L}_{\mathcal{M}}(\gamma(t)) = ||\gamma(t)-\hat{x}(t)||_2 + ||G_C ^{(\mathcal{M})}(\hat{x}(t)) \dot{\gamma}(t)||_2$$

