**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 5

**Dates:** 8-2 to 8-8

## Goals
- Reproduce results of Oozeer et al (2026)
- Fully debug geodesic computation
- Evaluate isometry on interlayer maps

## Approach and Implementation
I began this week by first trying to reproduce the results of Oozeer et al (2026), which visualize a curved geodesic between the representations of "Sunday" and "Wednesday" (see Notes) in the last-token residual stream space after the 28th layer of the model we have been working with -- the metric this curve is a geodesic wrt is important and discussed later. The weekday modality considers questions of the form `What day is {number} days after {entity}?`. I started building the pipeline by first configuring `5` paraphrases of this template and generated `245` (5 * 49) datapoints. These datapoints were then wrapped in Llama 3.1's chat template and then split into an 80/20 train-test split with balanced answer classes. 

The layer 28 representations are sourced from the train split and used to fit a PCA(64) subspace, where the dataset is further reduced to 7 centroids aggregate by ground-truth answer. We choose two such centroids, $p$ and $q$, and uniformly choose a set of carrier prompts, seeded per-pair and not sharing the ground-truth answer of either centroid's representative weekday, from the test split. The unwrapped rest-of-network map is then constructed, including the transformation $p \rightarrow \sqrt{p}$ (see log 4). The geodesics $\gamma _{p \rightarrow q} ^{(\text{carrier})}$ are then computed per-carrier via L-BFGS (ablation with AdamW is reported in Data). MSE and Pearson correlation between the computed geodesic and the linear baseline are reported as well as all optimization data. Geodesics are visualized in a PCA(2) subspace fit using the 4096 dimensional, reconstructed centroids from the PCA(64) subspace to avoid representing drift in the geodesic from directions in the orthogonal complement of the PCA(64) subspace. 

The above is the result of a series of iterations of the pipeline. Initial results were poor -- gradients were tiny, geodesics were identical to the linear baseline, and the singular values of the Jacobian at the point of maximum sensitivity along the linear baseline were non-uniform, meaning cheap directions existed at a waypoint (I defer this claim to work in Notes) but there was not enough signal to push the solver there. This was especially apparent when I injected spherical, Gaussian noise into the linear initialization; the solver struggled to converge, and only really did after several restarts. Initially, when optimization was done in the raw 4096 space, I took this as a sign that the solver struggled with the dimensionality. However, after adapting my code to work in PCA(64), the same symptoms surfaced. 

Eventually, I discovered that the rest-of-network map's outputs seldom put mass on any of the weekday token ids and, in fact, the top-1 token was often syntax related ('To', '\n ', etc). After aligning the model with the expected output tokens, which required changing the templates, the aforementioned symptoms subsided and genuine, curved geodesics finally started showing up. However, when I looked at the induced traces in the output probability simplex, I noticed that the top-1 probability at the endpoints didn't always match up with the ground-truth weekday. This finding was very sensitive to the carrier prompt. I decided to iterate on the work of the original authors here and instead of computing an expectation over carriers (which seemed to exacerbate this issue), I chose the carrier prompt which maintained the representational fidelity of the centroids.

$$\arg \min _{\text{carrier}} \space \lVert  \space D_{\text{KL}} (p ^{(\text{carrier})} (\text{centroids}) \space \lVert \space [I_7; 0]) \space  \rVert_1$$

The geodesics deviate from the linear baseline and show sensitivity to carrier prompt in the induced traces in the output probability simplex. The induced traces are unnatural and the transitions are not smooth, but do show a faster surfacing of the top-1 probability being the endpoint compared to the linear baseline. This is to be expected -- the pullback metric computes geodesics which induce, themselves, geodesics in the output probability simplex. These induced geodesics are not necessarily "natural", as seen in the work of Wurgaft et al, but arise as the result of the isometry of the rest-of-network map under the pullback metric. 

During the development of the pipeline, I managed to get into contact with Narmeen Oozeer, who gave me some insight about this. They claimed that the geodesics wrt the pullback should actually be straight, since the induced path is also straight in Hellinger coordinates (the metric in the output space is Euclidean in these coordinates). To get any kind of curvature, there would need to be an off-manifold penalty present in the loss. Their result in the aforementioned visualization is the geodesic wrt a learned surrogate to the pullback metric, whose curvature is due to approximation error. I had some questions about this, and we will meet to discuss further sometime this or next week.


## Results
- Curved geodesics for weekday modality realized and geodesic computation fully validated


## Notes
<img width="927" height="735" alt="image" src="https://github.com/user-attachments/assets/c7a1f02d-e91f-4fd9-b5e4-652083b77502" />

<img width="4032" height="3024" alt="IMG_1210" src="https://github.com/user-attachments/assets/8765f7a8-f1a5-4a72-b1aa-19b584a23d05" />


### Data
Carrier prompts often affect the representational fidelity of the centroids! Across 5 different carriers, the "Wednesday" centroid is mapped to a distribution whose mode is at "Sunday".
<img width="3048" height="2483" alt="image" src="https://github.com/user-attachments/assets/93efcd9e-a6ac-408c-ba05-2a4c56da585c" />
Choosing the carrier prompt which affects this the least yields the following:
<img width="1646" height="475" alt="image" src="https://github.com/user-attachments/assets/0e4df276-0a9d-4bc4-aa18-0e5da32744d1" />
Results using the final pipeline:
<img width="2272" height="1954" alt="image" src="https://github.com/user-attachments/assets/7e0b06de-3b52-4124-87fa-3bed53152115" />
<img width="926" height="488" alt="Recording 2026-08-07 153706" src="https://github.com/user-attachments/assets/526b8f7e-fbf3-4de6-a69f-82ab15527808" />





[Solver Data Across 10 Centroid Pairs with 1 Carrier Prompt using LBFGS/AdamW](https://claude.ai/code/artifact/48e1f1b2-7e6d-4b22-91f3-15648db146f3) (note, this is where carrier-sensitivity is especially present)

