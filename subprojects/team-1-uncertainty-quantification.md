# Uncertainty Quantification
The goal is to:
- Determine which uncertainty quantification method produces the most reliable, well-calibrated uncertainty estimates when the INR is being trained in the active learning loop, where the training distribution is not fixed but shaped by the uncertainty estimates themselves.

- Test whether the prior finding from passive reconstruction (MC Dropout is best calibrated) survives this feedback dynamic, and establish a cost-adjusted ranking of all UQ methods for practical deployment.

## Proposition

The [UncertaINR paper (Luo et al. 2023)](https://arxiv.org/abs/2202.10847) established that MC Dropout provides the best-calibrated uncertainty for INRs in a passive reconstruction setting, training data is fixed and sampled uniformly. Active learning setup might break this assumption. The training data is now selected based on the uncertainty estimates themselves: high-uncertainty coordinates are sampled, the model updates, and the uncertainty map changes. This creates a feedback loop absent from all prior UQ benchmarks for INRs.

**Research Question**: Is calibration quality observed in passive settings not a reliable predictor of calibration quality in the active loop? Methods that are slightly overconfident may cause the acquisition function to under-explore, concentrating samples in a shrinking set of flagged regions and leaving large areas unmodeled. Methods that are slightly underconfident may cause over-exploration, wasting budget on already-modeled regions. The feedback loop amplifies small miscalibrations over rounds.

## Proof of Concept
Here we describe a first deliverable that will be requested. Table 1 outlines the experimental parameters for this initial run.

| Parameter | Details |
| :--- | :--- |
| **Dataset** | Three images from Kodak (768×512) |
| **Architecture** | SIREN |
| **Acquisition** | MaxVar |
| **Budget** | $k_0=64$, $k=64$, $T=10$, 3 seeds |
| **UQ** | MCDrop, Evidential DL |

## Methods Compared

| Method | Description | Compute cost |
|--------|-------------|--------------|
| **MC Dropout** | 10 stochastic forward passes with dropout at test time. Variance across passes = epistemic uncertainty. Lighter dropout, more passes. Tests hyperparameter sensitivity of the default. | 10-20× forward pass |
| **Deep Ensemble** | N Independent INRs from different seeds. Disagreement = uncertainty. | N× training cost |
| **Evidential DL** | Output layer predicts ($\gamma$, $\nu$, $\alpha$, $\beta$) of a Normal-Inverse-Gamma distribution. Epistemic uncertainty = $\beta/(\nu(\alpha-1))$. Single forward pass. | 1× forward pass |
| **Last-Layer Laplace** | Gaussian approximation to the posterior over the final linear layer using the empirical Fisher Information Matrix. Analytically tractable. | 1× train + Hessian |
| **Gaussian Process** | GP with predictive variance. Closed-form posterior. Calibration gold standard from scientific computing. MaxVar Acquisition. | O($n^3$) per update |

Methods related to other subprojects remain fixed.

## Datasets
We'll work on these generally available datasets:

### Image Reconstruction

| Dataset | Details |
|---------|---------|
| Kodak (subset of the 24 images) | 768×512, RGB. Standard INR benchmark. |
| DIV2K validation (subset of the 100 images) | ~2K resolution. Subset of 10 images for higher-resolution generality check. |
| Scientific images (subset of the 10 images) | Microscopy, satellite, spectral at ~512². Tests transfer beyond natural photographs. |

### Scientific Discovery

| Dataset | Details |
|---------|---------|
| Branin-Hoo (2D) | Three global minima. Smooth multimodal. Standard BO benchmark. |
| Ackley (2D, 4D, 6D) | Flat outer region with sharp central minimum. Dimension sweep tests calibration scaling. |
| Rastrigin (2D, 4D) | Regularly spaced local minima. Tests calibration in highly multimodal fields. |
| Rosenbrock (2D, 4D) | Narrow curved valley. Tests calibration in elongated anisotropic structures. |
| Gaussian Random Fields (synthetic) | Controlled length scale. Systematic variation of smoothness and heterogeneity. |


## Evaluation
Outline of the metrics used to evaluate the model's performance, specifically focusing on signal reconstruction quality, uncertainty quantification calibration, and active learning efficiency. 

---

### 1. Calibration Reliability Diagram
A visual tool used to determine if a model's predicted confidence ($\sigma$) aligns with its empirical accuracy. A perfectly calibrated model forms a 45-degree diagonal line; deviations indicate overconfidence or underconfidence. Plotting this at rounds 1, 5, and 10 visualizes how self-awareness evolves during the MaxVar acquisition process.

**Calculation:**
General method for plotting:
1. Group predictions into $M$ interval bins based on predicted confidence.
2. Calculate the average predicted confidence $\text{conf}(B_m)$ within each bin:
$\text{conf}(B_m)=\frac{1}{|B_m|}\sum_{i \in B_m}\hat{p}_i$
3. Calculate the actual accuracy $\text{acc}(B_m)$ for the samples in that bin:
$\text{acc}(B_m)=\frac{1}{|B_m|}\sum_{i \in B_m}\mathbf{1}(|\hat{y}_i-y_i| \leq \tau)$
4. Plot accuracy (y-axis) vs. confidence (x-axis).

**Parameters:**
* $M$: Total number of bins the probability space is divided into.
* $B_m$: The set of pixel samples from the Kodak images whose predicted confidence ($\sigma$) falls into bin $m$.
* $|B_m|$: The total number of pixel samples in that specific bin.
* $\hat{p}_i$: The SIREN model's predicted confidence probability for pixel $i$.
* $\hat{y}_i$: The model's predicted color/intensity value for pixel $i$.
* $y_i$: The true value of pixel $i$ from the Kodak image.
* $\mathbf{1}(...)$: An indicator function that equals 1 if the prediction is correct (within an acceptable tolerance $\tau$) and 0 otherwise.
* $\tau$: The acceptable error threshold for a continuous regression prediction to be considered "correct".

**Resources:** Concept: [Scikit-Learn: Probability Calibration](https://scikit-learn.org/stable/modules/calibration.html). Implementation: [sklearn.calibration.calibration_curve](https://scikit-learn.org/stable/modules/generated/sklearn.calibration.calibration_curve.html)

---

### 2. Expected Calibration Error (ECE)
The numerical summary of the reliability diagram. It calculates the weighted average of the absolute difference between the predicted confidence and the true accuracy across all the bins. A lower ECE means the model is highly accurate at judging when its reconstructed pixels are likely to be wrong.

$$\text{ECE}=\sum_{m=1}^M\left[\frac{|B_m|}{N}|\text{acc}(B_m)-\text{conf}(B_m)|\right]$$

$\text{acc}(B_m)$ and $\text{conf}(B_m)$ are calculated using the specific equations defined in Section 1.

**Parameters:**
* $M$: Total number of bins.
* $N$: Total number of pixel samples across all bins (e.g., the cumulative samples acquired up to round $T$).
* $B_m$: The set of pixel samples falling into bin $m$.
* $|B_m|$: The count of samples in bin $m$.

**Resources:** Concept: [Obtaining Well Calibrated Probabilities (Naeini et al.)](https://aaai.org/Papers/AAAI/2015/AAAI-NaeiniM.pdf). Implementation: [torchmetrics.classification.CalibrationError](https://lightning.ai/docs/torchmetrics/stable/classification/calibration_error.html)

---

### 3. PSNR / NRMSE vs Cumulative Samples
Standard signal processing benchmarks for measuring the raw quality of the reconstructed image. PSNR measures image clarity, while NRMSE measures the pixel-level difference between the predicted image and the true Kodak image. Plotting these against cumulative samples maps the learning curve over the $T=10$ rounds.

First, calculate the Mean Squared Error (MSE):
$$\text{MSE}=\frac{1}{n}\sum_{i=1}^n(Y_i-\hat{Y}_i)^2$$

Then, calculate the metrics:
$$\text{PSNR}=10\cdot\log_{10}\left(\frac{\text{MAX}_I^2}{\text{MSE}}\right)$$

$$\text{NRMSE}=\frac{\sqrt{\text{MSE}}}{y_{\max}-y_{\min}}$$

**Parameters:**
* $n$: Total number of pixels in the image (e.g., 768 × 512).
* $Y_i$: The true color/intensity value of pixel $i$ in the original Kodak image.
* $\hat{Y}_i$: The reconstructed value of pixel $i$ generated by the model.
* $\text{MAX}_I$: The maximum possible pixel value (e.g., 255 for standard 8-bit images, or 1.0 if the image is normalized).
* $y_{\max}$, $y_{\min}$: The absolute maximum and minimum pixel values present in the true image.

**Resources:** Concept: [GeeksforGeeks: PSNR](https://www.geeksforgeeks.org/python/python-peak-signal-to-noise-ratio-psnr/). Implementation: [skimage.metrics.peak_signal_noise_ratio](https://scikit-image.org/docs/stable/api/skimage.metrics.html#skimage.metrics.peak_signal_noise_ratio)

---

### 4. Calibration Drift
A measure of whether the model loses its self-awareness as it learns. It evaluates the stability of the UQ estimates by comparing the Expected Calibration Error at the start of the experiment to the end, revealing if the ingestion of $k=64$ new points per round systematically skews the model's confidence.

$$\Delta\text{ECE}=\text{ECE}_{T=10}-\text{ECE}_{T=1}$$

$\text{ECE}$ is calculated using the formulas defined in Section 2.

**Parameters:**
* $\text{ECE}_{T=10}$: The Expected Calibration Error calculated at the final active learning round.
* $\text{ECE}_{T=1}$: The Expected Calibration Error calculated at the initial active learning round (using just the $k_0=64$ base samples).

**Resources:** Derived manually using outputs from Section 2.

---

### 5. Cost-Adjusted Ranking
An efficiency metric that balances accuracy against computational expense. It allows you to directly compare computationally heavy methods against lightweight ones by normalizing their calibration performance against a shared compute baseline.

$$\text{Cost-Adjusted Score}=\frac{\text{ECE}_{\text{target}}}{\text{ECE}_{\text{baseline}}}\times\frac{\text{Cost}_{\text{target}}}{\text{Cost}_{\text{baseline}}}$$

$\text{ECE}$ is calculated using the formulas defined in Section 2.

**Parameters:**
* $\text{ECE}_{\text{target}}$: The ECE of the specific UQ method you are evaluating (e.g., Evidential DL).
* $\text{ECE}_{\text{baseline}}$: The ECE of your reference UQ method (e.g., MCDrop).
* $\text{Cost}_{\text{target}}$: The computational expense of the evaluated method (e.g., 1 forward pass for Evidential DL).
* $\text{Cost}_{\text{baseline}}$: The computational expense of the baseline method (e.g., multiple forward passes for MCDrop).

**Resources:** Derived manually using outputs from Section 2.

---

### 6. Acquisition-Error Correlation
A test to evaluate the efficacy of the MaxVar data acquisition strategy. It uses a Spearman rank to check if the unobserved pixels the model is most uncertain about are genuinely the pixels with the largest actual errors, validating that the budget is spent on highly informative data points. 

Spearman's $\rho$:
$$\rho=1-\frac{6\sum_{i=1}^n d_i^2}{n(n^2-1)}$$

Where the rank difference is defined as:
$d_i=\text{Rank}(\sigma_i)-\text{Rank}(|Y_i-\hat{Y}_i|)$

**Parameters:**
* $\rho$: Spearman's rank correlation coefficient (closer to 1.0 means your UQ is successfully guiding the model to the worst mistakes).
* $\sigma_i$: The SIREN model's predicted uncertainty (variance) for unobserved pixel $i$.
* $|Y_i-\hat{Y}_i|$: The absolute true error of the model's prediction for pixel $i$ against the Kodak baseline.
* $d_i$: The difference between the predicted uncertainty rank and the true error rank for pixel $i$.
* $n$: The total number of unobserved pixels being evaluated for the next acquisition step.

**Resources:** Concept: [Spearman Rank Correlation](https://www.ncbi.nlm.nih.gov/books/NBK559156/). Implementation: [scipy.stats.spearmanr](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html)