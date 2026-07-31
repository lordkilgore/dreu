**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 4

**Dates:** 7-26 to 8-1

## Goals
- Compute Layer Pullback
- Implement Geodesic Solver
- Evaluate Isometry

## Approach and Implementation
A large portion of my time this week was spent revisiting the literature on the technical details of manifold steering. What I misunderstood previously is that the manifold is latent and not constructed. In Wurgaft et. al (2026), the authors explicitly estimate the manifold using a spline parameterization and show that the geodesics wrt the pullback from behavior space to activation space follow the spline quite well. In Ooozer et. al (2026), this observation is formalized; the geodesics wrt this pullback metric correspond to geodesics in the behavior space, so the framework should be to estimate this metric, then find the geodesic. In essence, their approach recapitulates the approach of Wurgaft et. al in reverse. Other approaches exist to estimate the manifold directly using activation density (Isomap, UMAP, t-SNE), however the literature regarding these methods lacks evidence of any of the nice isometric properties that the pullback-induced geodesics exhibit on the behavior space. This motivates the framing of manifold steering as first pullback metric estimation, then geodesic traversal.

For this project, we examine the pullback metric on the layer level. A layer $\ell \in [L]$ of a decoder block admits a representation $X_\ell \in \mathbb{R}^{K \times d_\text{model}}$ of our $K$ variable value centroids. Within the representation manifold $\mathbb{R}^{d_\text{model}}$, we posit, by the manifold hypothesis, the existence of a submanifold $\mathcal{M}_\ell \subset \mathbb{R}^{d_{\text{model}}}$, with intrinsic dimension $d$, that interpolates $\mathcal{X}_\ell$. Following recent work, we claim that geodesics under the pullback $J_{\Phi_\ell} ^T J_{\Phi_{ell}}$ from layer $\ell$ to layer $\ell + 1$ follow the model's learnt representation manifold, which should hence approximate the true data manifold $\mathcal{M}_\ell$.


## Results



## Notes


