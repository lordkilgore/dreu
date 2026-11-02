# Week 8

**Dates:** 08-23 to 08-29

## Goals
- Try preliminary methods (1) and (2) to select $\lambda_A$ and $\lambda_B$ hyperparameters


## Approach and Implementation
The sweep showed that hand-picked $`(\lambda_{a}, \lambda_{b})`$ don't transfer across modalities: in month and age, the behavior manifold is about $`10^{-3}`$ times the scale of the activation manifold, so $`\mathcal{L}_{\mathcal{B}}`$ barely registers. This week I derived two principled ways to set the weights.

*Gaussian position–velocity residuals.* Treating each manifold's stacked position and velocity residuals as spherical Gaussian noise turns the composite loss into a negative log-likelihood, with each weight playing the role of a precision. The precisions have closed-form maximum-likelihood estimates,

$$
\lambda_{\mathcal{Z}}^{*} = \frac{D_{\mathcal{Z}}}{2\,\overline{\mathcal{L}_{\mathcal{Z}}}},
$$

and plugging them back in gives a weight-free objective, $`D_{\mathcal{A}} \log \overline{\mathcal{L}_{\mathcal{A}}} + D_{\mathcal{B}} \log \overline{\mathcal{L}_{\mathcal{B}}}`$. With $`(D_{\mathcal{A}}, D_{\mathcal{B}}) = (128, 16)`$ this is an 8:1 weighting of the log-losses.

*Scaled-isometry forcing.* I fit a scale $`\hat{c}`$ by least squares so that on-manifold geodesic distances along $`F(\mathcal{A})`$ match $`\hat{c}`$ times those along $`\mathcal{A}`$, sampling point pairs uniformly from $`\mathcal{A}`$. I then set $`(\lambda_{\mathcal{A}}, \lambda_{\mathcal{B}}) = (\hat{c}^{2}, 1)`$ so that the squared residuals of the two terms are on a common scale.


## Results
- Both heuristics remove the hand-tuned weights and depend only on quantities already computed during optimization.
- The Gaussian objective rescales automatically: a loss term with a small typical value gets a large weight. Results are noisy and unclear to make any immediate conclusion about.
- The Gaussian heuristic assumes residuals along a manifold are independent and identically distributed across waypoints, which is fairly strong; scaled-isometry forcing is only as good as the isometry assumption it encodes.
- Scaled-isometry forcing reproduces the work of Wurgaft et al. but has a less dramatic effect on other modalities.

## Notes


