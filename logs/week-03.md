**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 3

**Dates:** 07-19 to 07-25

## Goals
- Analyze Lens Results
- Perform Layer Identification
- Visualize Candidate Layer Representations

## Approach and Implementation
Debugging the logit lens implemented last week, I was finally able to analyze the results. What I observed was uninterpretable, high-entropy output distributions using earlier layer
representations and uninformative, low-entropy output distributions using later layer representations. In effect, the transition between these distributions was diffusive in nature
and ultimately did not allow for any way to identify "shape" with one layer and "angle" with another layer.

Sidestepping, I turned to a parallel approach called **activation patching**, which patches the last-token representation of a prompt at a given layer with another prompt's last-token representation, with the only difference in the prompts being the value of exactly one variable. This methodology aims to measure at what layer a variable becomes causally load-bearing. The effect of patching was measured using the binary logit difference between answer classes (namely "yes" and "no" to the template `"A {SHAPE} is placed on a table which has a {ANGLE} degree decline. Will the object slide off? Answer with exactly one word: yes or no."`), KL divergence in output distributions, and KL divergence restricted to the distribution over answer classes. The effect is aggregate within a variable strata using a fixed layer's representations and recorded. For more details on this procedure, see the notes section below.

Across all 3 metrics, the effect for both variables was mostly uniform across variable strata. Averaging out this stratum axis, the inflection for both variables occurred at layer 15. Repeating the same experiment with the template `"A {} is rotated {} degrees about an axis through opposite vertices. Is this transformation a symmetry of the shape? Answer with exactly one word: yes or no."` yielded the same results.

Two variables surfacing at the same layer is a negative result in the context of this experiment and I hypothesize the following explanations:
- *Poor Layer Conditioning*: Rest-of-network maps at later layers have steeper Lipschitz behavior; patching at these layers lead to large perturbations in the output due solely to the map's analytic behavior. Confirming this hypothesis would require estimating the Lipschitz constants of these maps on the data, but overall is not worth exploring for the purpose of this project. It seems like this is pretty confounding for activation patching in general.
- *Insufficient Resolution*: Circuits responsible for shape/angle surfacing are sublayer or interlayer (like induction heads). This is out of scope, since our steering experiments use representations on the layer level.

Sidestepping once more, I turned to the literature and found that the type of identification I was trying to make could be found in the arithmetic data modality. In particular, "Addition in Four Movements: Mapping Layer-wise Information Trajectories in LLMs" (Yan, 2025) shows that the formulaic structure of binary addition is decodable by linear probe at roughly layer 16 and onwards of Llama-3-8B. Motivated by this, I repeated the same experiments with the template `"Calculate: {x} + {y}= "`, replacing the binary logit difference and answer KL metrics with a greedy logit difference, which measures difference in top-1 logits between a patched and clean prompt. 

The results using the KL divergence metric showed the same stable stratification across layers as before, but only for $x$. For the greedy logit difference, the results were much noisier, presumably due to inherited model noise, and there was no clear stratification. Averaging out the variable stratum axis, both metrics agreed in inflection at layer 30 for $x$ and layer 14/15 for $y$. Uncertainties for $y$ in these estimates were significant and inverse variance weighting led to less interpretable estimates. SEM was calculated via method-of-moments (total variance decomposition with $E[effect \mid stratum ; layer]$) and effect error was calculated via sample variance within each (layer, stratum) sample.

Noise in the KL divergence data for this experiment was also observed in the shape/angle experiments. Refining this approach would involve computing the ground-truth labels for each prompt and using that for a logit difference metric, ideally reducing the observed variance.

Finally, I visualized the layer 14 ($y$) and layer 30 ($x$) representations (and their respective class centroids) using PCA.


## Results
- Logit lens on shape/angle modality led to uninterpretable results.
- Activation patching on shape/angle modality led to negative results, identifying both variables with layer 15.
- Activation patching on arithmetic modality led to positive results, identifying $x$ with layer 30 and $y$ with layer 14 (albeit with significant uncertainty in the latter).
- Representations of arithmetic modality variables visualized using PCA.


## Notes

### Activation Patching
Let $\mathcal{G}$ denote the variable set. For every $X \in \mathcal{G}$, we look at the collection $\mathcal{A}_X$ of prompt sets where every variable besides $X$ is held fixed. Let $A \in \mathcal{A}_X$ be one of such prompt sets and consider the set $A^{2*} = A^2 \setminus \{(P, P) : P \in A\}$.

For every pair of prompts $(P, Q) \in A^{2*}$, cache the residual stream at the last position for $P$ at each layer in $R_P[i]$ and the logits associated with the last position, $logits_Q$ and $logits_{P}$. For each layer $i$, replace $R_Q\[i\]$ with $R_P\[i\]$, record $logits_{P\rightarrow Q} \[i\]$ by letting the forward pass finish.

The effect of the patch is a normalized score that measures how much the answer has been influenced by $P$ and is measured per-layer:

$$effect_{ (P,Q) }[i] = \frac{d_{P \rightarrow Q}[i] - d_Q}{d_P - d_Q}$$

$$d_R = (logits_{R})_{yes} - (logits_{R})_{no}$$

This metric is then aggregate over all pairings in $A^{2*}$ to estimate how much patching moves toward the donor's answer within a variable stratum (particularly of $\mathcal{G} \setminus \{X\}$):

$$effect[i] = E_{(P,Q)} \[ effect_{P\rightarrow Q}\[i\]\]$$

In our experiments, we take $\mathcal{G} = \{SHAPE, ANGLE\}$ and use this methodology to compute the $effect[ANGLE][layer]$ and $effect[SHAPE][layer]$ heatmaps for the shape and angle variables respectively.

### Data

<img width="1592" height="575" alt="image" src="https://github.com/user-attachments/assets/df92429e-1e43-48af-a46d-155ad1887712" />

<img width="1700" height="615" alt="image" src="https://github.com/user-attachments/assets/697cbe54-ab29-434f-aede-7b2daaf4787e" />

<img width="1676" height="1327" alt="image" src="https://github.com/user-attachments/assets/44bcc11d-e193-4bcd-98f3-512de9157ea7" />

<img width="1714" height="1290" alt="image" src="https://github.com/user-attachments/assets/39b747b2-2346-48f6-ad98-6b2de5f84f92" />

<img width="1335" height="1109" alt="image" src="https://github.com/user-attachments/assets/0405faf4-6c45-4267-9686-e96be9a1a469" />


