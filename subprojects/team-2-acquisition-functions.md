# Acquisition Functions

The goal is to:
- Determine which acquisition strategy produces the most sample-efficient active learning loop (i.e., achieving the best reconstruction quality for the fewest oracle queries).

- Compare four principled active strategies against five baselines, ranging from random sampling to the theoretical upper bound. 

- Characterise when information-theoretic criteria outperform simple variance sampling, when batch diversity matters, and whether any active strategy consistently beats Sobol (the strongest passive baseline).

## Proposition

MaxVar (sampling wherever the model is most uncertain) is already principled and surprisingly competitive. The core question is whether more sophisticated strategies justify their additional complexity and computational overhead.

Two orthogonal improvements over MaxVar are possible: 
1. **Information-theoretic grounding:** Rather than asking where the output is uncertain, ask which observations would most constrain the model's parameters. FisherEIG operationalises this via the D-optimal design criterion.

2. **Batch diversity:** MaxVar selects points independently, ignoring redundancy. If two high-uncertainty points are spatially close, sampling both is computationally wasteful. BatchDPP addresses this through determinantal batch selection.

This subproject tests whether either improvement is empirically significant and whether they interact with one another.

**Research Question**: Under what conditions do the empirical gains of information-theoretic grounding and batch diversity in active learning justify their computational complexity and overhead over standard maximum variance and Sobol sampling.

## Proof of Concept

Here we describe a first deliverable that will be requested. Table 1 outlines the experimental parameters for this initial validation run.

| Parameter | Details |
| :--- | :--- |
| **Dataset** | Branin-Hoo (2D) |
| **Architecture** | SIREN |
| **Acquisition** | Random, MaxVar, Sobol |
| **Budget** | $k_0=16$, $k=16$, $T=10$, 3 seeds |
| **UQ** | MCDrop |


## Methods Compared

### Active Strategies

| Strategy | Description | Key Test |
| :--- | :--- | :--- |
| **MaxVar** | Top-k coordinates by $\sigma^2(x)$. Per-point evaluation, no batch coordination. | Default benchmark. All improvements measured against this. |
| **FisherEIG** | Maximise log-det of Fisher Information Matrix (D-optimal). Greedy submodular selection with $(1-1/e)$ guarantee. Requires LaplaceLL for analytical FIM. | Is D-optimal worth the compute overhead over A-optimal? |
| **BatchDPP** | Determinantal Point Process: kernel $L_{ij} = \sigma(x_i)\sigma(x_j) \cdot \text{sim}(\phi(x_i),\phi(x_j))$. Quality-diversity tradeoff via $\alpha$ parameter. | Does batch diversity outperform greedy top-k, and when? |
| **GradAcq** | Expected gradient length: $4 \cdot \sigma^2(x) \cdot \|\nabla_\theta f(x)\|^2$. Targets uncertain coordinates where new data would most move the parameters. | Does gradient influence complement uncertainty? |

### Baselines

| Strategy | Description | Key Test |
| :--- | :--- | :--- |
| **Random** | $k$ coordinates selected uniformly at random. No model, no uncertainty. | Null hypothesis. Every active method must beat this. |
| **Sobol** | Low-discrepancy quasi-random sequence. $O((\log N)^d/N)$ discrepancy. Non-adaptive. | Strongest passive baseline. Hard to beat when the signal is uniform. |
| **Farthest Point Sampling** | Greedily select the coordinate farthest from all observed points. Pure spatial coverage. | Does uncertainty add anything beyond spatial coverage? |
| **Adaptive Grid** | Coarse uniform grid; refine cells where uncertainty exceeds threshold via quadtree/octree. | Simple engineering baseline. Must be beaten to justify continuous optimisation. |
| **Oracle Error** | Select $k$ coordinates with the highest true reconstruction error. Impossible in practice. | Theoretical ceiling. The gap to this measures how much better UQ could theoretically get. |


<!-- ## Ablation: Budget Granularity

Fixing the total budget at $B=1024$, this ablation varies the division between batch size ($k$) and the number of active learning rounds ($T$) to identify the sweet spot where additional rounds stop providing value relative to the retraining cost.

| Configuration | Initial ($k_0$) | Batch ($k$) | Rounds ($T$) |
| :--- | :--- | :--- | :--- |
| **One-shot** (no active learning) | 1024 | — | 0 |
| **Coarse rounds** | 128 | 128 | 7 |
| **Medium rounds** | 64 | 64 | 15 |
| **Fine rounds** | 16 | 16 | 63 | -->

## Datasets

### Analytical Benchmarks

| Function | Details |
| :--- | :--- |
| **Branin-Hoo (2D)** | Three global minima. Smooth multimodal. The most widely cited BO benchmark. |
| **Ackley (2D, 4D, 6D)** | Flat outer region with a sharp central minimum. Tests high-dimensional scaling. |
| **Rastrigin (2D, 4D)** | Regularly spaced local minima. Tests global structure vs. local traps. |
| **Rosenbrock (2D, 4D)** | Narrow curved valley. Tests anisotropic structure handling. |
| **Shekel (4D)** | Multiple sharp peaks of varying height and width. |
| **Hartmann (3D, 6D)** | Multiple local optima with a known global optimum. Standard BO benchmark. |

### Physically Motivated Oracles

| Dataset | Details |
| :--- | :--- |
| **Materials Project DFT** | API-based. Formation energy or band gap as a function of alloy composition. Directly relevant to materials discovery. |
| **CALPHAD phase diagram** | ESPEI-based. Binary alloy phase stability as a function of composition and temperature. Natural test for sharp boundary-detection. |
| **Gaussian Random Fields** | Synthetic. Controlled length scale and smoothness for systematic signal regularity analysis. |
<!-- | **PINN surrogate** | PDE-based field (heat diffusion/stress) served as an instantaneous oracle by a pretrained PINN. Physical structure without per-query FEM cost. | -->


## Evaluation

Outline of the metrics used to evaluate the acquisition strategies' performance, specifically focusing on sample efficiency, reconstruction quality, and the behavior of the active learning loop.

*(Note: Diagnostic outputs such as acquisition heatmaps, BatchDPP $\alpha$ sweeps, and the FisherEIG greedy gap will be captured visually alongside these core quantitative metrics).*

---

### 1. NRMSE / PSNR vs Cumulative Oracle Queries
Standard signal processing benchmarks for measuring the raw quality of the reconstructed signal, plotted against the cumulative acquisition budget. This maps the primary learning curve for each strategy, allowing visual comparison of sample efficiency (visualised as one line per strategy, $\pm1$ standard deviation shaded).

**Calculation:**
First, calculate the Mean Squared Error (MSE):
$$\text{MSE}=\frac{1}{n}\sum_{i=1}^n(Y_i-\hat{Y}_i)^2$$

Then, calculate the metrics:
$$\text{PSNR}=10\cdot\log_{10}\left(\frac{\text{MAX}_I^2}{\text{MSE}}\right)$$

$$\text{NRMSE}=\frac{\sqrt{\text{MSE}}}{y_{\max}-y_{\min}}$$

**Parameters:**
* $n$: Total number of unobserved coordinates in the target field.
* $Y_i$: The true value of coordinate $i$ from the oracle.
* $\hat{Y}_i$: The reconstructed value of coordinate $i$ generated by the SIREN architecture.
* $\text{MAX}_I$: The maximum possible signal value.
* $y_{\max}$, $y_{\min}$: The absolute maximum and minimum values present in the true signal.

**Resources:** Concept: [GeeksforGeeks: PSNR](https://www.geeksforgeeks.org/python/python-peak-signal-to-noise-ratio-psnr/). Implementation: [skimage.metrics.peak_signal_noise_ratio](https://scikit-image.org/docs/stable/api/skimage.metrics.html#skimage.metrics.peak_signal_noise_ratio)

---

### 2. AUC-Budget (Area Under the Curve)
Measures the total area under the PSNR-vs-budget learning curve, normalised by the theoretical Oracle Error baseline. This metric captures overall performance across all active learning rounds, penalising strategies that learn slowly even if they eventually catch up.

**Calculation:**
$$\text{AUC-Budget}=\frac{\sum_{t=1}^T \text{PSNR}_{\text{target}}(t) \cdot k}{\text{AUC}_{\text{oracle}}}$$

**Parameters:**
* $T$: Total number of active learning rounds.
* $k$: Number of samples acquired per round (budget granularity).
* $\text{PSNR}_{\text{target}}(t)$: The PSNR achieved by the evaluated strategy at round $t$.
* $\text{AUC}_{\text{oracle}}$: The area under the curve achieved by the theoretical Oracle Error baseline.

**Resources:** Derived manually by integrating the area under the outputs from Section 1.

---

### 3. Sample Efficiency Ratio
Quantifies exactly how much faster an active strategy reaches a specific target reconstruction quality compared to the Random baseline.

**Calculation:**
$$\text{Efficiency Ratio}=\frac{B_{\text{Random}}(\text{target})}{B_{\text{Strategy}}(\text{target})}$$

**Parameters:**
* $\text{target}$: The specific PSNR or NRMSE threshold being evaluated.
* $B_{\text{Random}}(\text{target})$: The cumulative budget (number of samples) required by the Random baseline to achieve the target.
* $B_{\text{Strategy}}(\text{target})$: The cumulative budget required by the evaluated active strategy to achieve the target.

**Resources:** Derived manually by thresholding the learning curves from Section 1.

---

### 4. Acquisition-Error Spatial Correlation
A diagnostic test to evaluate if the acquisition function successfully targets regions with the highest actual errors. It uses a Spearman rank correlation between the acquisition score and the true error at unobserved coordinates.

**Calculation:**
Spearman's $\rho$:
$$\rho=1-\frac{6\sum_{i=1}^n d_i^2}{n(n^2-1)}$$

Where the rank difference is defined as:
$$d_i=\text{Rank}(A(x_i))-\text{Rank}(|Y_i-\hat{Y}_i|)$$

**Parameters:**
* $\rho$: Spearman's rank correlation coefficient.
* $A(x_i)$: The acquisition score assigned to unobserved coordinate $x_i$ (e.g., predicted variance, expected gradient length).
* $|Y_i-\hat{Y}_i|$: The absolute true error of the model's prediction for coordinate $i$.
* $d_i$: The difference between the acquisition score rank and the true error rank for coordinate $i$.
* $n$: The total number of unobserved coordinates evaluated for the next step.

**Resources:** Concept: [Spearman Rank Correlation](https://www.ncbi.nlm.nih.gov/books/NBK559156/). Implementation: [scipy.stats.spearmanr](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html)

