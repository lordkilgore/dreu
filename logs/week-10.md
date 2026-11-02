# Week 10

**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

**Dates:** 10-4 to 10-10

## Goals
- Finalize report


## Approach and Implementation
This week I finalized the report, focusing on the figures and on tightening the writing.

*Figures.* I regenerated every plot from the experiment code instead of reusing notebook screenshots, exporting each as a PDF with matching fonts and line widths so they scale cleanly in LaTeX. The final set is:
- the per-layer KL patching curves for the two arithmetic variables,
- side-by-side confinement geodesics at $`\lambda_T = 0`$ and $`\lambda_T = 0.05`$, to show the waypoint-jump fix,
- a 2×2 grid of per-waypoint loss variance, grouped by $`\lambda_a`$ and by $`\lambda_b`$ for both loss terms.

I also adjusted float placement so each figure and table appears after the text that first refers to it.

*Wording and technical details.* I went through the paper for precision:
- Corrected the claim that lifted spline paths are pullback geodesics. They aren't: pullback geodesics straighten the image in output space rather than following $`\mathcal{B}`$.
- Stated the null-space bound on $`J_F`$ exactly, rather than as an estimate.
- Fixed the implicit-function derivation so the numerator is the vector $`r'(t^*)^\top`$, and justified dropping the distance-potential term.
- Made the notation consistent throughout: norms, the Chamfer definition, and the measures $`\mu`$ and $`\nu`$.
- Cut redundant passages and fixed the citation keys so the bibliography compiles cleanly.


## Results
- A finished report: abstract, problem formulation, related work, method, results and discussion, with final figures and tables.
- Every technical claim now has either a derivation or a pointer to the experiment that supports it. The limitations (one model, mainly one modality, single-carrier evaluation) are stated explicitly.


## Notes


