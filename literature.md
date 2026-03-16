# Literature: Active Coordinate Sampling for INR World Models

Welcome! This document contains 26 papers that together form the theoretical and methodological foundation of our research project. Your task is to pick one paper, read it thoroughly, remake one experiment and prepare a presentation for the group.

The papers are sorted from most accessible to most demanding in terms of conceptual prerequisites. If you are newer to machine learning, consider choosing from the top of the list. Papers near the bottom assume familiarity with concepts introduced in earlier ones, so reading a few of the foundational entries first will help regardless of which paper you present.

When preparing your presentation, focus on three things: (1) what problem does the paper solve, (2) what is the key idea, and (3) what are the main results. The descriptions below give you a starting point.

If you have questions about any paper or want help scoping your presentation, don't hesitate to reach out.

---

**[1]** · **Implicit Neural Representations with Periodic Activation Functions** · ★☆☆☆☆
Vincent Sitzmann, Julien N. P. Martel, Alexander W. Bergman, David B. Lindell, Gordon Wetzstein. *NeurIPS 2020.*

SIREN is an MLP that uses sinusoidal activation functions instead of ReLU, paired with a principled weight initialization scheme. This allows the network to accurately fit continuous signals such as images, audio, and 3D shapes from coordinate-value pairs, while also representing their spatial derivatives.

SIREN is one of the most widely used INR architectures and serves as a common backbone that we will build on later.

---

**[2]** · **On the Spectral Bias of Neural Networks** · ★☆☆☆☆
Nasim Rahaman, Aristide Baratin, Devansh Arpit, Felix Draxler, Min Lin, Fred A. Hamprecht, Yoshua Bengio, Aaron Courville. *ICML 2019.*

This paper demonstrates empirically and theoretically that deep ReLU networks learn low-frequency components of a target function before high-frequency ones. The phenomenon, called spectral bias, means that standard MLPs struggle to represent fine detail unless given enough training time or architectural modifications.

Spectral bias has direct implications for where an INR will make errors, which matters when deciding where to collect new observations.

---

**[3]** · **Random Features for Large-Scale Kernel Machines** · ★☆☆☆☆
Ali Rahimi, Benjamin Recht. *NeurIPS 2007.* Test of Time Award 2017.

Proposes mapping input data to a randomized low-dimensional feature space using random Fourier bases, so that inner products in the new space approximate those of a shift-invariant kernel. The construction is grounded in Bochner's theorem, which states that any positive-definite shift-invariant kernel is the Fourier transform of a non-negative measure.

This paper is the mathematical foundation that Fourier feature encodings for INRs [5] build upon. Understanding the connection between random feature maps and kernel approximation clarifies why Fourier input mappings help neural networks represent high-frequency content.

---

**[4]** · **Dropout as a Bayesian Approximation: Representing Model Uncertainty in Deep Learning** · ★★☆☆☆
Yarin Gal, Zoubin Ghahramani. *ICML 2016.*

Shows that applying dropout at test time and running multiple stochastic forward passes is mathematically equivalent to approximate variational Bayesian inference. The variance across forward passes provides an estimate of epistemic uncertainty, meaning regions where the model lacks data will show high disagreement between passes.

MC Dropout is one of the main uncertainty quantification methods we will use.

---

**[5]** · **Fourier Features Let Networks Learn High Frequency Functions in Low Dimensional Domains** · ★★☆☆☆
Matthew Tancik, Pratul P. Srinivasan, Ben Mildenhall, Sara Fridovich-Keil, Nithin Raghavan, Utkarsh Singhal, Ravi Ramamoorthi, Jonathan T. Barron, Ren Ng. *NeurIPS 2020.*

Demonstrates that mapping low-dimensional inputs through random Fourier features before passing them to a ReLU MLP overcomes spectral bias and enables the network to represent high-frequency content. The paper includes a Neural Tangent Kernel analysis showing how the encoding bandwidth controls the network's frequency response.

Fourier feature encodings are a widely used alternative to sinusoidal activations for controlling what frequencies an INR can capture.

---

**[6]** · **Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles** · ★★☆☆☆
Balaji Lakshminarayanan, Alexander Pritzel, Charles Blundell. *NeurIPS 2017.*

Proposes training multiple independently initialized copies of the same network on the same data and using their disagreement as an uncertainty measure. Each copy converges to a different local minimum, and points where these minima produce different predictions are likely underspecified by the data.

Deep Ensembles are a standard uncertainty quantification baseline and one of the methods used in our pipeline.

---

**[7]** · **Occupancy Networks: Learning 3D Reconstruction in Function Space** · ★★☆☆☆
Lars Mescheder, Michael Oechsle, Michael Niemeyer, Sebastian Nowozin, Andreas Geiger. *CVPR 2019.*

Represents 3D surfaces as the continuous decision boundary of a neural network that takes a 3D coordinate as input and classifies it as inside or outside a shape. This was one of the first papers to establish that neural networks can serve as compact, continuous, resolution-independent representations of geometry.

The coordinate-to-occupancy formulation is directly relevant to environment field reconstruction tasks.

---

**[8]** · **DeepSDF: Learning Continuous Signed Distance Functions for Shape Representation** · ★★☆☆☆
Jeong Joon Park, Peter Florence, Julian Straub, Richard Newcombe, Steven Lovegrove. *CVPR 2019.*

Learns continuous signed distance fields for 3D shapes using an auto-decoder architecture in which per-shape latent codes and network weights are jointly optimized without an encoder. This allows a single shared network to represent many different shapes, each identified by its latent code.

The auto-decoder paradigm introduced here is the foundation for latent-conditioned INR architectures used in later work.

---

**[9]** · **Instant Neural Graphics Primitives with a Multiresolution Hash Encoding** · ★★☆☆☆
Thomas Müller, Alex Evans, Christoph Schied, Alexander Keller. *ACM Transactions on Graphics (SIGGRAPH) 2022.* Best Paper Award.

Replaces learned positional encodings with a multiresolution hash table of trainable feature vectors. Each resolution level divides space into a grid whose vertices store small feature vectors, and non-vertex coordinates are interpolated trilinearly. Hash collisions are resolved implicitly by the MLP, reducing INR training time from minutes to seconds.

This is the basis for the fast hash-encoded INR backbone we will use alongside SIREN.

---

**[10]** · **BACON: Band-Limited Coordinate Networks for Multiscale Scene Representation** · ★★☆☆☆
David B. Lindell, Dave Van Veen, Jeong Joon Park, Gordon Wetzstein. *CVPR 2022.*

Introduces a coordinate network architecture with an analytical Fourier spectrum, where each output layer has a provably bounded frequency bandwidth. By controlling the bandwidth at each layer, BACON learns a band-limited multiscale decomposition of the signal without per-scale supervision, and its behaviour at unobserved points is predictable from its spectral design.

BACON makes the relationship between network architecture and frequency content explicit. This is relevant because the frequency content of reconstruction errors determines where active sampling should focus, and an architecture with controllable spectral properties makes that interaction analysable.

---

**[11]** · **NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis** · ★★★☆☆
Ben Mildenhall, Pratul P. Srinivasan, Matthew Tancik, Jonathan T. Barron, Ravi Ramamoorthi, Ren Ng. *ECCV 2020.*

Maps 5D coordinates (3D position plus 2D viewing direction) to volume density and radiance via an MLP with positional encoding, then renders novel views by integrating colour and density along camera rays using differentiable volumetric rendering.

NeRF established the dominant paradigm for neural scene representation and generated the active view-selection literature that several later papers on this list belong to.

---

**[12]** · **Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks** · ★★★☆☆
Chelsea Finn, Pieter Abbeel, Sergey Levine. *ICML 2017.*

Introduces MAML, a bi-level optimization algorithm that learns an initialization of network weights such that a few gradient steps on a new task yield strong performance. The outer loop updates the initialization across a distribution of tasks, and the inner loop adapts it to each specific task.

MAML is the standard framework for learning good initializations and will be applied to INRs for fast adaptation from sparse observations.

---

**[13]** · **Bayesian Active Learning for Classification and Preference Learning** · ★★★☆☆
Neil Houlsby, Ferenc Huszár, Zoubin Ghahramani, Máté Lengyel. *arXiv:1112.5745, 2011.*

Derives the mutual information between model predictions and model parameters as the optimal acquisition function for active learning, an approach the authors call Bayesian Active Learning by Disagreement (BALD). This quantity equals the difference between the predictive entropy and the expected entropy under the posterior, and it can be estimated using MC Dropout samples.

BALD is the foundational information-theoretic acquisition criterion that most modern batch active learning methods build upon, including several of our proposed acquisition functions.

---

**[14]** · **BatchBALD: Efficient and Diverse Batch Acquisition for Deep Bayesian Active Learning** · ★★★☆☆
Andreas Kirsch, Joost van Amersfoort, Yarin Gal. *NeurIPS 2019.*

Extends BALD from single-point to batch acquisition by computing the joint mutual information between a batch of query points and the model parameters. A greedy selection algorithm with a (1-1/e) submodularity guarantee prevents the redundancy that arises when selecting points independently.

This paper formalises why batch-aware selection outperforms picking the top-k individually most uncertain points, which is the same problem our batch acquisition strategies address.

---

**[15]** · **Learned Initializations for Optimizing Coordinate-Based Neural Representations** · ★★★☆☆
Matthew Tancik, Ben Mildenhall, Terrance Wang, Divi Schmidt, Pratul P. Srinivasan, Jonathan T. Barron, Ren Ng. *CVPR 2021.*

Applies MAML and Reptile to learn weight initializations for coordinate-based networks across distributions of signals including images, CT scans, and 3D shapes. The resulting initializations converge orders of magnitude faster than random initialization and generalize better from partial observations.

This is a direct demonstration of meta-learning applied to INRs and is relevant to settings where rapid fitting from few samples matters.

---

**[16]** · **Deep Evidential Regression** · ★★★☆☆
Alexander Amini, Wilko Schwarting, Ava Soleimany, Daniela Rus. *NeurIPS 2020.*

Modifies a regression network to output the four parameters of a Normal Inverse-Gamma distribution, which serves as a higher-order prior over the predicted mean and variance. This separates aleatoric and epistemic uncertainty in a single forward pass without requiring ensembles or multiple stochastic passes.

Evidential regression offers the cheapest per-query uncertainty estimation among the methods considered in our work.

---

**[17]** · **UncertaINR: Uncertainty Quantification of End-to-End Implicit Neural Representations for Computed Tomography** · ★★★☆☆
Francisca Vasconcelos, Bobby He, Nalini Singh, Yee Whye Teh. *Transactions on Machine Learning Research (TMLR) 2023.*

Benchmarks four uncertainty quantification methods (MC Dropout, deep ensembles, Hamiltonian Monte Carlo, and Bayes-by-Backprop) applied to INRs for CT reconstruction. The main finding is that MC Dropout achieves the best calibration for coordinate-based networks, outperforming even full Bayesian inference via HMC.

This is the most directly relevant study of UQ quality specifically for INR architectures.

---

**[18]** · **Laplace Redux: Effortless Bayesian Deep Learning** · ★★★☆☆
Erik Daxberger, Agustinus Kristiadi, Alexander Immer, Runa Eschenhagen, Matthias Bauer, Philipp Hennig. *NeurIPS 2021.*

Provides a unified framework and open-source library for post-hoc Laplace approximation of neural network posteriors. After standard training, the method fits a Gaussian approximation to the posterior using the Hessian (or Fisher information matrix) of the loss at the converged weights. The last-layer variant is particularly efficient.

The Laplace approximation produces the same Fisher information object needed for information-theoretic acquisition functions, making it serve a dual role in our pipeline.

---

**[19]** · **From Data to Functa: Your Data Point Is a Function and You Can Treat It Like One** · ★★★☆☆
Emilien Dupont, Hyunjik Kim, S. M. Ali Eslami, Danilo Jimenez Rezende, Dan Rosenbaum. *ICML 2022.*

Represents each data point as a modulated SIREN, where a shared INR backbone is conditioned on per-instance modulation vectors optimized via auto-decoding. Downstream tasks like generation, classification, and imputation are then performed directly on these functional representations.

The modulated-SIREN architecture enables a single network to represent many different signals and is the basis for generalizable INR models that must handle diverse environments.

---

**[20]** · **3D Gaussian Splatting for Real-Time Radiance Field Rendering** · ★★★☆☆
Bernhard Kerbl, Georgios Kopanas, Thomas Leimkühler, George Drettakis. *ACM Transactions on Graphics (SIGGRAPH) 2023.* Best Paper Award.

Represents scenes as collections of anisotropic 3D Gaussians with learned opacity, colour, and covariance, rendered via differentiable rasterization rather than ray marching. This explicit representation achieves real-time rendering at quality comparable to NeRF.

3D Gaussian Splatting has become the dominant alternative to NeRF for neural scene representation and is the primary representation used by FisherRF [23] for active view selection.

---

**[21]** · **ActiveNeRF: Learning where to See with Uncertainty Estimation** · ★★★★☆
Xuran Pan, Zihang Lai, Shiji Song, Gao Huang. *ECCV 2022.*

Augments NeRF with learned per-ray variance outputs and uses the predicted uncertainty to select which camera views to acquire next. Training views that maximally reduce expected uncertainty are chosen, and the results show significant reconstruction improvements under limited input budgets.

This was the first paper to formalise active learning for neural radiance fields and is a direct precursor to coordinate-level active sampling.

---

**[22]** · **Bayes' Rays: Uncertainty Quantification for Neural Radiance Fields** · ★★★★☆
Lily Goli, Cody Reading, Silvia Sellán, Alec Jacobson, Andrea Tagliasacchi. *CVPR 2024.*

Applies Laplace approximation to a pre-trained NeRF by learning a spatial perturbation field over the input coordinates, producing volumetric uncertainty maps without modifying the original training procedure. The perturbation-based approach avoids retraining and yields well-calibrated spatial uncertainty estimates.

This demonstrates that post-hoc Bayesian methods can provide useful uncertainty fields for neural representations.

---

**[23]** · **FisherRF: Active View Selection and Mapping with Radiance Fields Using Fisher Information** · ★★★★☆
Wen Jiang, Boshu Lei, Kostas Daniilidis. *ECCV 2024 (Oral).*

Computes the Fisher information matrix over radiance field parameters (3D Gaussian Splatting and Plenoxels) and selects the next training view by maximizing expected information gain via D-optimal experimental design. The greedy selection algorithm comes with a submodularity-based approximation guarantee.

This is the closest existing work to coordinate-level Fisher-information-based active sampling and provides the theoretical blueprint for extending the approach from views to individual coordinates.

---

**[24]** · **Neural Ordinary Differential Equations** · ★★★★☆
Ricky T. Q. Chen, Yulia Rubanova, Jesse Bettencourt, David Duvenaud. *NeurIPS 2018.* Best Paper Award.

Replaces discrete residual-network layers with continuous ODE dynamics parameterized by a neural network, using an adaptive ODE solver for the forward pass and the adjoint sensitivity method for memory-efficient backpropagation. This enables modelling of continuous-time processes with constant memory cost regardless of depth.

Neural ODEs are foundational for any neural world model that must capture temporal evolution alongside spatial structure.

---

**[25]** · **Physics-Informed Neural Networks: A Deep Learning Framework for Solving Forward and Inverse Problems Involving Nonlinear Partial Differential Equations** · ★★★★★
Maziar Raissi, Paris Perdikaris, George E. Karniadakis. *Journal of Computational Physics, Vol. 378, 2019.*

Trains coordinate-based neural networks with PDE residuals as soft constraints in the loss function, enabling simultaneous data fitting and physics enforcement for solving differential equations governing fluid dynamics, quantum mechanics, and other physical systems.

PINNs establish the paradigm of using coordinate-based networks as physics-constrained surrogates, which is relevant to scientific domains where observations are expensive and sampling locations must be chosen carefully.

---

**[26]** · **Continuous PDE Dynamics Forecasting with Implicit Neural Representations (DINo)** · ★★★★★
Yuan Yin, Matthieu Kirchmeyer, Jean-Yves Franceschi, Alain Rakotomamonjy, Patrick Gallinari. *ICLR 2023 (Spotlight).*

Encodes spatiotemporal PDE states into modulated-INR latent representations driven by learned neural ODEs, enabling forecasting at arbitrary spatial and temporal resolutions including extrapolation beyond training grids.

DINo integrates INRs, neural ODEs, and auto-decoding into a unified world-model framework for physical dynamics and represents the most advanced combination of concepts covered by this reading list.

---

## Quick-reference table

| # | Topic Label | Paper (short) | Difficulty |
|---|------------|---------------|------------|
| 1 | INR Architecture | SIREN | ★☆☆☆☆ |
| 2 | Spectral Bias | Spectral Bias of NNs | ★☆☆☆☆ |
| 3 | Input Encoding (Theory) | Random Fourier Features | ★☆☆☆☆ |
| 4 | Uncertainty Quantification | MC Dropout | ★★☆☆☆ |
| 5 | Input Encoding | Fourier Features for INRs | ★★☆☆☆ |
| 6 | Uncertainty Quantification | Deep Ensembles | ★★☆☆☆ |
| 7 | INR Architecture (3D) | Occupancy Networks | ★★☆☆☆ |
| 8 | INR Architecture (3D) | DeepSDF | ★★☆☆☆ |
| 9 | INR Architecture (Fast) | Instant-NGP | ★★☆☆☆ |
| 10 | INR Architecture (Multiscale) | BACON | ★★☆☆☆ |
| 11 | Neural Scene Representation | NeRF | ★★★☆☆ |
| 12 | Meta-Learning | MAML | ★★★☆☆ |
| 13 | Active Learning (Info-Theoretic) | BALD | ★★★☆☆ |
| 14 | Batch Active Learning | BatchBALD | ★★★☆☆ |
| 15 | Meta-Learning for INRs | Learned Initializations | ★★★☆☆ |
| 16 | Uncertainty Quantification | Evidential Regression | ★★★☆☆ |
| 17 | UQ for INRs | UncertaINR | ★★★☆☆ |
| 18 | Uncertainty Quantification | Laplace Redux | ★★★☆☆ |
| 19 | Meta-Learning for INRs | Functa | ★★★☆☆ |
| 20 | Neural Scene Repr. (Explicit) | 3D Gaussian Splatting | ★★★☆☆ |
| 21 | Active Learning for NFs | ActiveNeRF | ★★★★☆ |
| 22 | UQ for Neural Fields | Bayes' Rays | ★★★★☆ |
| 23 | Active View Selection | FisherRF | ★★★★☆ |
| 24 | Neural Dynamics | Neural ODEs | ★★★★☆ |
| 25 | Scientific INRs / PINNs | PINNs | ★★★★★ |
| 26 | Scientific INRs / PDE | DINo | ★★★★★ |
