
## Abstract

The nervous system represents time dependent signals in sequences of discrete, identical action potentials or spikes; information is carried only in the spike arrival times. We show how to quantify this information, in bits, free from any assumptions about which features of the spike train or input signal are most important, and we apply this approach to the analysis of experiments on a motion sensitive neuron in the fly visual system. This neuron transmits information about the visual stimulus at rates of up to 90 bits/s, within a factor of 2 of the physical limit set by the entropy of the spike train itself.

## Breakdown

- In response to a long sample of time varying stimuli, the spike train of a single neuron varies, and we can quantify this variability by the entropy per unit of time of the spike train, $S(\Delta \tau)$, which depends on the time resolution $\Delta \tau$ with which we record the spike arrival times.
- Because of stochasticity, if we repeat the same exact time dependent stimulus, we see a similar, but no precisely identical, sequence of spikes. This variability at fixed input can also be quantified by entropy, which we call the conditional or noise entropy per unit time $\mathcal{N}(\Delta\tau)$. 
- The information that the spike train provides about the stimulus is the difference between these entropies, $R_{\text{info}} = S - \mathcal{N}$.
- Because $\mathcal{N}$ is positive semi-definite, $S$ sets the capacity for transmitting information. We can define an efficiency metric $\epsilon(\Delta \tau) = R_{\text{info}}(\Delta \tau) / S(\Delta \tau)$ with which this capacity is used.
- The question of whether spike timing is important is really the question of whether this efficiency is high at small $\Delta\tau$.
- The goal of this paper is to give a completely model independent estimate of entropy and information in neural spike trains as they encode dynamic signals.
- We begin by discretizing the spike train into time bins of size $\Delta \tau$.
- We examine segments of the spike train in windows of length $T$ so that each possible neural response is a "word" with $T /\Delta\tau$ symbols.
- Let us call the normalized count of the $i$th word $\tilde p_i$. Then the "naive estimate" of the entropy is $$ S_{\text{naive}}(T, \Delta\tau; \text{size}) = -\sum_i \tilde p_i \log_2 \tilde p_i $$
	- Note that our estimate depends on the size of the dataset.
- The true entropy is $$S(T, \Delta\tau) = \lim_{\text{size}\to \infty} S_{\text{naive}}(T, \Delta;\text{size})$$
- We are interested in the entropy rate $S(\Delta \tau) = \lim_{T\to\infty} S(T, \Delta\tau)/ T$. At large $T$, very large data sets are required to ensure convergence of $S_{\text{naive}}$ to the true entropy $S$.
- Imagine a spike train with mean spike rate of $\tilde r \sim 40$ spikes/s, sampled with a time resolution $\Delta\tau = 3$ ms.
	- In a window of $T = 100$ ms, the maximum entropy consistent with the mean rate is $S\sim 17.8$ bits, and the entropy of real spike trains is not far from this bound.
	- Entropy is measured in bits, where 1 bit corresponds to a binary question (yes/no).
	- An entropy of $S = 17.8$ bits means that, on average, it would take **17.8 binary questions** (yes/no questions) to determine which specific pattern (word) occurred in the neuron's spike train for that time window.
	- Then there are roughly $2^{S} \sim 2\times 10^5$ words with significant $p_i$, and our naive estimation procedure seems to require that we observe many samples of each word.
		- This is because $2^S$ is the number of distinct states you can represent with $S$ number of bits. Think of it like asking "How many different words are possible if I have $S$ binary choices?"
	- If our samples come from nonoverlapping 100 ms windows, then we need much more than 3 hours of data.
		- To estimate entropy accurately, they need many samples of each word, implying they need a large dataset (more than 3 hours of data if using non-overlapping 100 ms windows).
- First, we examine explicitly the dependence of $S_{\text{naive}}$ on the size of the dataset and find regular behaviors that can be extrapolated to the infinite data limit.
	- The authors find that the naive entropy exhibits regular behavior as a function of dataset size. Thus, it changes in a predictable way when more data is added.
	- They use this to extrapolate to the infinite data limit. By studying how the naive entropy behaves with datasets size, they can make predictions about the true entropy without needing infinite data, which is impractical.
- Second, we evaluate robust bounds on the entropy that serve as a check on our extrapolation procedure.
	- Robust bounds are a reliable way to get upper and lower bounds on the true entropy. This allows us to conduct a sanity check to confirm that the estimation procedures are within a plausible range of the true entropy. These provie confidence in our estimate.
	- If they are tight, it suggests your estimate is accurate. If they are wide, it suggests uncertainty in the estimate.
- Third, we are interested in the extensive component of entropy, and we find that a clean approach to extensivity is visible before sampling problems set in.
	- An extensive quantity in physics is one that scales linearly with the size of the system. In this context, the entropy should scale linearly with the length of the time window $T$.
	- This is important because it ensures that we are capturing the underlying structure of the neural code.
- Finally, for the neuron studied - the motion sensitive neuron H1 in the fly's visual system - we can actually collect many hours of data.
- H1 responds to motion across the entire visual field, producing more spikes for an inward horizontal motion and fewer spikes for an outward motion; vertical motions are coded by other neurons.
- These cells provide visual feedback for flight control. In the experiments analyzed here, the fly is immobilized and views computer generated images on a display oscilloscope. For simplicity these images consists of vertical stripes with randomly chosen grey levels, and this pattern takes a random walk in the horizontal direction.
- We begin our analysis with time bins of size $\Delta \tau = 3$ ms. For a window of $T = 30$ ms - corresponding to the behavioral response time of the fly - Figure 2 shows the histogram $\{\tilde p_i\}$, and the naive entropy estimates. 
- We see that there are very small finite dataset corrections ($<10^{-3}$), well fit by $$S_{\text{naive}}(T, \Delta\tau;\text{size}) = S(T, \Delta) + \frac{S_1(T, \Delta)}{\text{size}} + \frac{S_2(T, \Delta)}{\text{size}^2}$$
	- This equation comes form "The Upward Bias in Measures of Information Derived from Limited Data Samples" A. Treves, S. Panzeri.
	- When working with empirical data, estimates of information measures are often biased upward due to the finite size of the data samples.
	- This is equation comes from statistical theories of entropy estimation. When you estimate probabilities from a a finite dataset, the entropy estimate typically has corrections that decay with the size of the dataset.
- Under these conditions, we feel confident that the extrapolated $S(T, \Delta)$ is the correct entropy.
	- The authors emphasize that the finite dataset corrections are small because:
		- The dataset they are working with is large.
		- The word probabilities are well-sampled (even rare words have been observed enough times).
- For sufficiently large $T$, finite size corrections are larger, the contribution of the second correction is significant, and the extrapolation to infinite size is unreliable.
	- Finite size corrections refer to the biases or errors in entropy estimates caused by having a finite dataset.
	- As $T$ increases, the window's size used to define words becomes longer.
		- The number of possible spike pattern grows exponentially with $T$, $2^{\frac{T}{\Delta\tau}}$. 
		- For larger $T$, many of these possible words occur infrequently or not at all leader to:
			- Larger sampling biases.
			- Larger finite size corrections because rare or unseen words are poorly estimated.
- Ma discussed the problem of entropy estimation in the undersampled limit. For probability distributions that are uniform on a set of $N$ bins (as in the microcanonical ensemble), the entropy is $\log_2(N)$ and the problem is to estimate $N$. Ma noted that this could be done by counting the number of times that two randomly chosen observations yield the same configuration, since the probability of such a coincidence is $1/N$.
	- In the undersampled limit, the dataset is not large enough to reliably estimate all possible probabilities $p_i$. This can lead to biases in entropy estimation because
		- Rare events might not appear at all.
		- The naive entropy estimate becomes unreliable.
		- In statistical mechanics, the microcanonical ensemble describes a system where all configurations (states) are equally likely. If there are $N$ possible states, the true entropy is $S = \log_2 N$, and so the challenge becomes estimating $N$.
		- The probability of a coincidence in a uniform distribution is $1/N$. Thus, by counting coincidences is $1/N$. Thus, by counting coincidences, $N$ (and hence $S$) can be inferred.
- More generally, the probability of a coincidence is $P_c = \sum p_i^2$, and hence $$S = -\sum p_i \log_2 p_i = -<\log_2 p_i> \geq -\log_2 <p_i> = -\log_2 P_c$$ so we can compute a lower bound to the entropy by counting coincidences. Furthermore, $\log_2 P_c$ is less sensitive to sampling errors than $S_{\text{naive}}$. The Ma bound is one of the Renyi entropies widely used in the analysis of dynamical systems.
	- The probability of a coincidence occurring is the sum of all possible coincidences (we get the same $i$ state twice) $$P_c = \sum p_i^2$$
	- If one state $i$ dominates the distribution (high concentration), $P_c$ approaches $1$.
	- The arithmetic mean-geometric mean (AM-GM) inequality states $$<x> \geq GM(x)$$ where $<x>$ is the arithmetic mean is $\frac{\sum_{i=1}^n a_i}{n}$ and the geometric mean $GM(x)$ is $\sqrt[n]{a_1\cdots a_n}$  .
	- $S = -E[\log_2(p_i)]$ which is estimated by $S \approx -<\log_2(p_i)>$ 
	- Recall the log-sum inequality, which states that $<\log_2 p_i > \leq \log_2 <p_i>$. 
	- Thus, $$-<\log_2 p_i > \geq - \log_2 <p_i>$$
	- Therefore, $$S \geq -\log_2 P_c$$ since $S = -E[\log_2(p_i)] \geq -\log_2 E[p_i] = -\log_2 P_c$, where $$E[p_i] = \sum_i p_i \cdot p_i = \sum_i p_i^2 = P_c$$
	- $P_c$ summarizes the concentration of $\{p_i\}$: higher $P_c$ implies a more concentrated distribution.
	- $-\log_2 P_c$ provides a minimum uncertainty estimate based on $P_c$. The entropy $S$ can NOT be smaller than this value.
	- This is related to the Renyi entropy of order $2$, which emphasizes the concentration of probabilities and is widely used in the analysis of dynamical systems.
- The Ma bound is tightest for distributions that are close to uniform, but distributions of neural responses cannot be uniform because spikes are sparse. The distribution of words with fixed spike count is more nearly uniform, so we use $S \geq S_{\text{Ma}}$, with $$S_{\text{Ma}} = -\sum_{N_{sp}} P(N_{sp})\times \log_2 \Bigg[P(N_{sp}) \frac{2n_c(N_{sp})}{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]} \Bigg]$$ where $n_c(N_{sp})$ is the number of coincidences observed among the words with $N_{sp}$ spikes, $N_{obs}(N_{sp})$ is the total number of occurrences of words with $N_{sp}$ spikes, and $P(N_{sp})$ is the fraction of words with $N_{sp}$ spikes.
	- The Ma bound assumes the probability distribution is close to uniform. However, neural spike trains are not uniform because spikes are sparse and occur rarely.
	- To address this, the authors consider subsets of the data where all outcomes ("words") have the same number of spikes. These subsets of words are more nearly uniform, making the Ma bound more applicable.
	- $S_{Ma}$ aggregates over all subsets of words grouped by their spike counts $N_{sp}$.
	- $P(N_{sp})$ is the probability of observing a word with $N_{sp}$ spikes, relative to all words.
	- Within each group of words that have the same spike count $N_{sp}$, the entropy estimate depends on the coincidences among those words.
	- $n_c(N_{sp})$ is the number of coincidences observed among words with $N_{sp}$ spikes.
	- $N_{obs}(N_{sp})$ is the total number of observations of words with $N_{sp}$ spikes.
	- Consider the number of all possible pairs from a group of size $N_{obs}(N_{sp})$ number of observations. The number of ways to choose $2$ observations as the pair is given by $N_{obs}(N_{sp})\choose{2}$ $= \frac{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]}{2}$ 
	- The coincidence probability within this group is then $$\frac{n_c(N_{sp})}{\binom{N_{obs}(N_{sp})}{2}} = \frac{2n_c(N_{sp})}{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]}$$
	- Since $P(N_{sp})$ is the probability of a word have $N_{sp}$ spikes, the adjusted probability of coincidences for words with $N_{sp}$ spikes normalized by the probability of observing such words is: $$P(N_{sp}) \cdot \frac{2n_c(N_{sp})}{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]}$$
	-  For each $N_{sp}$ subset, the entropy contribution is $$-\log_2\Big[P(N_{sp}) \cdot \frac{2n_c(N_{sp})}{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]} \Big]$$ this measures the uncertainty associated with this subset of words.
	- Thus, the total entropy bound $S_{Ma}$ sums these contributions over all possible spike counts $N_{sp}$: $$S_{Ma} = -\sum_{N_{sp}}P(N_{sp})\times \log_2\Bigg[P(N_{sp}) \cdot \frac{2n_c(N_{sp})}{N_{obs}(N_{sp})[N_{obs}(N_{sp}) - 1]} \Bigg]$$
	- In Figure 3, we plot our entropy estimate as a function of the window size $T$. For sufficiently large windows the naive procedure gives an answer smaller than the Ma bound, and hence the naive answer must be unreliable because it is more sensitive to sampling problems.
	- For smaller windows the lower bound and the naive estimate are never more than 10-15 percent apart. The point at which the naive estimate crashes into the Ma bound is also where the second correction in  $$S_{\text{naive}}(T, \Delta\tau;\text{size}) = S(T, \Delta) + \frac{S_1(T, \Delta)}{\text{size}} + \frac{S_2(T, \Delta)}{\text{size}^2}$$ becomes significant and we lose control over the extrapolation to the infinite data limit, $T\approx 100$ ms. Beyond this point the Ma bound becomes steadily less powerful.
	- If the correlations in the spike train have finite range, then the leading subextensive contribution to the entropy will be a constant $C(\Delta\tau)$, $$\frac{S(T, \Delta\tau)}{T} = S(\Delta\tau) + \frac{C(\Delta\tau)}{T} + \ldots$$
	- This behavior is seen clrearly in Figure 3, and emerges before the sampling disaster, providing an estimate of the entropy per unit time, $(S\Delta\tau = 3 ms) = 157 \pm 3$ bits/s.
	- The entropy approaches its extensive limit from above, so that $$\frac{1}{\Delta\tau}[S(T + \Delta\tau, \Delta\tau) - S(T, \Delta\tau)] \geq S(\Delta\tau)$$ for all window sizes $T$. This bound becomes progressively tighter at larger $T$, until sampling problems set in. In fact there is a broad plataeu (plus or minus $2.7$ percent) in the range $18 < T < 60$ ms, leading to $S \leq 157$ plus or minus $4$ bits/s, in excellent agreement with the extrapolation in Figure 3.
- The noise entropy per unit time $\mathcal{N}(\Delta\tau)$ measures the variability of the spike train in response to repeated presentations of the same varying input signal. Marking a particular time $t$ relative to the stimulus, we accumlate the frequencies of occurrence $\tilde p_i(t)$ of each word $i$ that begins at $t$, then form the naive estimate of the local noise entropy in the window from $t$ to $t+T$, $$N^{local}_{naive}(t, T, \Delta\tau;size) = -\sum_{i}\tilde p_i(t)\log_2\tilde p_i(t)$$
- The average rate of information transmission by the spike train depends on the noise entropy averaged over $t$, $$N_{naive}(T, \Delta\tau;size) = E[N^{loca}_{naive}(t, T, \Delta\tau;size)]_t$$. Then we analyze as before the extrapolation to large data set size and large $T$, with results shown in Figure 3. The difference between the two entropies in the information which the cell transmits, $R_{info}(\Delta\tau = 3 ms) = 78\pm 5$ bits/s, or $1.8\pm 0.1$ bits/spike.
- The analysis has been carried out over a range of time resolutions
***

## Key Concepts

1. Entropy of Spike Trains:
	1. **Entropy** measures the total variability or randomness in the neural responses. A high entropy value implies that the neuron's response is very unpredictable and could be encoding a lot of information.
	2. **Noise entropy** measures the variability in the neural response due to intrinsic noise - variability that isn't related to the actual input stimulus.
	3. **Signal entropy** is the variability in the spike train specifically tied to the input stimulus.
2. Mutual Information
	1. The paper aims to determine how much of the variability in spike trains is actually meaningful in terms of the stimulus. The metric they derive, mutual information, tells us how much of the entropy in a neural response can be attributed to differences in the stimuli rather than randomness.

## Summary

In this study

Think of a neuron like a telegraph sending a Morse code. The spike patterns are the code words. Entropy $S$ tells you how complex that code is - how many different patterns are realistically in use. If $S$ is high, there are many distinct messages that the neuron can send (like a telegraph with a large vocabulary). If $S$ is low, then the messages are simpler or more predictable.


## Notes on Slides

- 3ms is behavioral response time of fly
- Order corrected naive estimate line fit, authors use this to establish confidence in their extrapolated true entropy value.