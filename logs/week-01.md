Student: Phoenix Wilson
Mentor: Kristina Gligoric

# Week 1

**Dates:** 07-05 to 07-11

## Goals
- Organize twice weekly meetings with mentor
- Put together and read corpus of readings related to my interests in representation finetuning and manifold steering
- Narrow down on a specific research question, along with what problem it addresses and its significance
- List out 5 initial tasks to explore this quetstion

## Approach and Implementation
I met with Kristina on Tuesday for 30 minutes, where we were able to set up a twice weekly meeting time on Mondays and Wednesdays. In addition to this, Kristina helped me narrow done an avenue to explore, namely manifold steering, and helped me settle into the workspace. 

Following this meeting, I put together the following corpus of readings related to manifold steering, a novel (May 2026) approach to steering techniques in interpretability:
- Manifold Steering Reveals the Shared Geometry of Neural Network Representation and Behavior (Wurgaft et al., 2026)
- Riemannian-Manifold Steering: Geometry-Aware Generative Autoencoders for Label-Free Steering (Oozeer et al., 2026)
- What Would Non-Linear Features Actually Look Like? (Gorton, 2026)
- Structuring Sparsity: Block-Sparse Featurizers Capture Visual Concept Manifolds (Fel et al., 2026)
- Manifold-Guided Attention Steering (Li et al., 2026)
- Verbalizable Representations Form a Global Workspace in Language Models (Gurnee et al., 2026)

From this corpus, I read the contents of items 1, 2, and 3 completely and skimmed for the main ideas in the other papers. Wurgaft et al. (2026) introduces manifold steering as
a method for maintaining behavior-space fidelity by steering activations along geodesics across an activation manifold. These geodesics are computed with respect to a pullback metric $J_F ^T J_F$, where $F$ denotes the rest-of-network map from the activation space to the behavior space (output distribution space). 

In practice, traversing the computation graph to compute $J_F$ is intractable. Instead, the authors access this metric implicitly by directly optimizing activation paths to induce a target trajectory in behavior space. Oozeer et al. (2026) builds onto this work by instead accessing the metric through a learnt encoder $\phi$ in activation space whose (regularized) pullback $J_\phi ^T J_\phi + \varepsilon I$ approximates $J_F ^T J_F$. In both of these papers, the authors, along with Fel et al. (2026), argue that the success of manifold steering is further evidence that the Linear Representation Hypothesis, as supported by Gorton (2026), is a weak framework, that linear steering does not suffice, and that activation manifolds -- not linear functionals on activation space -- may be the correct atomic unit in mechanistic interpretability.

Motivated by this argument, I became interested in studying how these activation manifolds are transformed throughout transformer layers in LLMs. In particular, whether downstream layers preserve the geometry of upstream layer representations. 

## Results
- Documented notes with Obsidian from fully read papers and main ideas from skimmed papers.
- Formulated research questions: "When a manifold-compliant intervention is performed at Layer $L$, does it stay on the manifold at layers $\ell > L$? If it doesn't, how might the non-linearity in the MLP blocks twist this geometry? Does it decompose into higher-dimensional Minkowski sums as Fel et al. observed in vision models?"
- Created a list of tasks to start exploring the hypothesis:
  1. Build a Toy Dataset. This dataset will include template contexts with clear hierarchical variables (e.g. "The [SHAPE] rolls down the table at a [DEG] incline")
  2. Integrate Llama with Forward Hooks. This will allow us to track the residual stream at every layer in the model.
  3. Implement Lens. Lens let us identify layers where certain variables (e.g. shape, incline) surface.
  4. Fit GAGA Encoder. Given candidate layers $\mathcal{L}$, we fit a manifold $\mathcal{M}_x$ for each variable $x$.
  5. Evaluate Isometry. Qualitatively investigate whether steering along a manifold in an early layer in $\mathcal{L}$ steering along the manifold of a later layer in $\mathcal{L}$.

Overall, this week was very instructive and I learnt alot!

## Notes


