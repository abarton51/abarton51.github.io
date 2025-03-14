---
title: "Entropy and Information in Neural Spike Trains"
excerpt: "An in-depth analysis of the quantification of information in neural spike trains based on entropy measures."
permalink: /posts/2024/12/01/entropyneural.md/
collection: portfolio
---

This page is dedicated to my notes and interpretations of the paper titled [Entropy and Information in Neural Spike Trains](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.80.197) by Strong et al. The paper was published in 1998 and is a great paper in my opinion for learning about neural spike trains and applications of information theory in computational neuroscience.

## Abstract

The nervous system encodes time-dependent signals through sequences of discrete, identical action potentials or spikes. These spikes carry information solely in their arrival times. This paper demonstrates how to quantify such information, measured in bits, without assuming specific features of the spike train or input signal. By applying this approach to a motion-sensitive neuron in the fly visual system, the study reveals that this neuron can transmit up to 90 bits/s, which is within a factor of 2 of the physical limit set by the entropy of the spike train itself.

## Quantifying Information in Neural Spike Trains

### Key Concepts

1. **Entropy of Spike Trains (\$S(\Delta \tau)\$):**

   - The variability in spike train responses to a time-dependent stimulus is captured by the entropy per unit time. This depends on the resolution \$\Delta \tau\$ used to record spike arrival times.

2. **Noise Entropy (\$\mathcal{N}(\Delta \tau)\$):**

   - Repeated presentations of the same stimulus result in similar but not identical spike trains. The variability across these repetitions is quantified by the conditional or noise entropy per unit time.

3. **Information Rate (\$R\_{\text{info}}\$):**

   - The information carried by the spike train about the stimulus is the difference between the total entropy and noise entropy:

   - As \$\mathcal{N}\$ is positive semi-definite, \$S\$ sets the upper limit for information transmission.

4. **Efficiency (\$\epsilon(\Delta \tau)\$):**

   - Defined as the fraction of entropy used to transmit information: $\epsilon(\Delta \tau) = R_{\text{info}}(\Delta \tau) / S(\Delta \tau)$

   - Investigating \$\epsilon(\Delta \tau)\$ at small \$\Delta \tau\$ helps determine the importance of spike timing.

### Methodology

#### Discretizing Spike Trains

- The spike train is divided into time bins of size \$\Delta \tau\$. For a total duration \$T\$, the spike train is represented as "words" composed of \$T / \Delta \tau\$ symbols.

- Let \$\tilde{p}\_i\$ represent the normalized count of the \$i\$th word. The naive estimate of the entropy is:

- The true entropy is obtained by extrapolating to infinite data size:

#### Entropy Rate

- The entropy rate, representing the information per unit time, is defined as: \$S(\Delta \tau) = \lim\_{T\to\infty} S(T, \Delta\tau)/ T\$.

#### Example Calculations

- For a spike train with mean spike rate \$\tilde{r} \approx 40\$ spikes/s sampled at \$\Delta \tau = 3\$ ms and \$T = 100\$ ms:
  - Maximum entropy: \$S \sim 17.8\$ bits.
  - This implies \$2^{S} \sim 2 \times 10^5\$ distinguishable patterns.
  - More than 3 hours of data would be required to estimate entropy accurately for non-overlapping 100 ms windows.

#### Finite-Size Effects

- The naive entropy estimate exhibits systematic dependence on dataset size:

- Large datasets and robust extrapolation techniques ensure reliable entropy estimation.

## Application to Fly Motion-Sensitive Neuron (H1)

### Experimental Setup

- **Neuron Functionality:**

  - The H1 neuron encodes horizontal motion, increasing spike rates for inward motion and decreasing rates for outward motion.
  - Vertical motion is handled by other neurons.

- **Stimulus Design:**

  - Immobilized fly views a computer-generated vertical stripe pattern with random grey levels.
  - Stripes undergo random horizontal motion.

- **Recording Parameters:**

  - Spike arrival times sampled with \$\Delta \tau = 3\$ ms.
  - Behavioral response time of \$T = 30\$ ms.

### Results

- For sufficiently large \$T\$, corrections to naive entropy estimates become small, and finite-size corrections diminish.
- The H1 neuron achieves information rates of up to 90 bits/s, demonstrating near-maximal use of the spike train’s entropy capacity.

### Key Insights

1. **Efficiency of Coding:**

   - High efficiency (\$\epsilon \sim 0.9\$) indicates that spike timing contributes significantly to encoding motion information.

2. **Entropy Scaling:**

   - Entropy scales linearly with \$T\$, consistent with the structured nature of neural codes.

## Conclusion

This study introduces a model-independent framework for quantifying information in neural spike trains, overcoming challenges like finite dataset sizes and stochasticity. Applied to the H1 neuron in flies, the method reveals efficient encoding of motion information through precise spike timing, achieving near-maximal information rates relative to entropy limits. This approach provides a robust tool for analyzing dynamic signal representations in the nervous system.

