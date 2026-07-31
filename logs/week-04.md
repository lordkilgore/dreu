**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 4

**Dates:** 7-26 to 8-1

## Goals
- Compute Layer Pullback
- Implement Geodesic Solver
- Evaluate Isometry

## Approach and Implementation
A large portion of my time this week was spent revisiting the literature on the technical details of manifold steering. What I misunderstood previously is that the manifold is latent and not constructed. In Wurgaft et al. (2026), the authors explicitly estimate the manifold using a spline parameterization and show that the geodesics wrt the pullback from behavior space to activation space follow the spline quite well. In Ooozer et al. (2026), this observation is formalized; the geodesics wrt this pullback metric correspond to geodesics in the behavior space, so the framework should be to estimate this metric, then find the geodesic. In essence, their approach recapitulates the approach of Wurgaft et. al in reverse. Other approaches exist to estimate the manifold directly using activation density (Isomap, UMAP, t-SNE), however the literature regarding these methods lacks evidence of any of the nice isometric properties that the pullback-induced geodesics exhibit on the behavior space. This motivates the framing of manifold steering as first pullback metric estimation, then geodesic traversal.

For this project, we examine the pullback metric on the layer level. A layer $\ell \in [L]$ of a decoder block admits a representation $X_\ell \in \mathbb{R}^{K \times d_\text{model}}$ of our $K$ variable value centroids. Within the representation manifold $\mathbb{R}^{d_\text{model}}$, we posit, by the manifold hypothesis, the existence of a submanifold $M_\ell \subset \mathbb{R}^{d_{\text{model}}}$, with intrinsic dimension $d$, that interpolates $X_\ell$. Following recent work, we claim that geodesics under the pullback $J_{\Phi_\ell} ^T J_{\Phi_{\ell}}$ from layer $\ell$ to layer $\ell + 1$ follow the model's learnt representation manifold, which should hence approximate the true data manifold $M_\ell$.

Computing $J_{\Phi_\ell} ^T J_{\Phi_{\ell}}$ is intractable at the scale of Llama 3.1-8B's decoder layers, with each Jacobian requiring a full pass through the computation graph. Moreover, the metric $J_{\Phi_\ell} ^T J_{\Phi_{\ell}}$ is generally sub-Riemannian (PSD). To work around these issues, for every representation pair $p, q$, I instead compute:

$$\sqrt{||J_{\Phi_{\ell}}(p)(p-q)||_2 ^2 + \varepsilon ||p-q||_2 ^2}$$

In this form, the Jacobian computation is replaced with a JVP and the factorization halves the space complexity. The regularizer absorbs $J_{\Phi_{\ell}}$'s rank deficiency with $\varepsilon$ set to `1e-3`. Implementing this programmatically was difficult, as Llama decoder layers expect lots of metadata about the input and the self-attention layers require a supplied context, not an individual vector. The standard workaround to this is to select a 'carrier prompt', with which the last token representation at a specified layer is patched with the representation of choice, say $p$. This spliced carrier prompt's $L$ by $d_{\text{model}}$ representation is then passed into the next-layer map and the JVP at $p$ (with another argument) is produced on the associated backward pass of this map.

Once the metric was implemented, I integrated `torch.optimizer.lbfgs` and used the hyperparameters used in Ooozer et al. (2026). The solver optimizes a path $\{\pi_k \}_{k=1} ^{K}$ of least length with respect to the regularized pullback, with $\pi_1$ and $\pi_K$ fixed to specified representations as endpoints of the geodesic. Using the regularized pullback as the loss is nonstandard however; not only does letting gradients flow through the Jacobian term incur memory overhead, but it can also introduce spurious descent directions that arise from the first-order approximation error in $ds \approx J dx$. For this reason, we detach the Jacobian term from the computation graph, coined the 'freeze-metric' adaptation, and let gradients flow solely through the difference term:

$$L(\pi) = \sum _{k=2} ^{K} \sqrt{||J_{\Phi_{\ell}}([\pi_k]_{\text{detached}})(\pi_k-\pi_{k-1})||_2 ^2 + \varepsilon ||\pi_k-\pi_{k-1}||_2 ^2}$$

Optimizing for geodesics with respect to the pullback metrics on the respective representation spaces for the $x$, $y$, and `shape` variables, the results were pretty surprising. Nearly all of the geodesics were straight line paths between centroids, with some curvature apparent in some paths. Ablating the Euclidean term in the loss, the paths remained the same and further analysis showed the pullback error consistently decreasing across all geodesic optimizations. Additionally, JVP computation matched full-Jacobian computation and freeze-metric adaptation was validated. Using `AdamW` as the solver yielded the same geodesics. It is clear that the solver is not the issue, so ruling out other possible functional failure modes is my next step (e.g. maybe prompt slicing is causing this). 

If it is truly the case that the geodesics are linear across both the arithmetic and shape modalities, it would be pretty surprising. By the chain rule, this would suggest that reproducing the results of Oozeer et al. (2026) with the pullback of the 28th layer from the output space would also lead to linear geodesics, not rounded ones. Verifying that the geodesics are linear with the weekday dataset would confirm that my code is functionally incorrect.

## Results
- Integrated JVP computation with freeze-metric adaptation.
- Implemented L-BFGS and AdamW optimizers for path-level geodesic optimization
- Visualized linear geodesics, in the process of debugging possible failure modes

## Notes

### Data
<img width="2600" height="1530" alt="image" src="https://github.com/user-attachments/assets/054c9e49-015a-4282-8147-d2442c918421" />


