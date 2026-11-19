---
tags:
  - mathematics/probability
  - intermediate
  - distributions
  - statistics
  - obsidian-vault
aliases:
  - Random Variables and Distributions
---

# Probability and Statistics: Random Variables and Distributions

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[STAT_01_Probability_Axioms_and_Bayes_Theorem]], [[STAT_03_Maximum_Likelihood_Estimation]]

---

## 1. Random Variables, Expectation, and Variance

A **random variable** $X$ is a measurable function mapping outcomes from a sample space to real numbers ($\Omega \to \mathbb{R}$).

### Mathematical Notation
* **Expected Value ($\mathbb{E}[X]$):** The theoretical mean or long-run average of $X$.
  $$\mathbb{E}[X] = \sum_{x} x P(X = x) \quad \text{(Discrete)}$$
  $$\mathbb{E}[X] = \int_{-\infty}^{\infty} x f(x) \, dx \quad \text{(Continuous)}$$

* **Variance ($\text{Var}(X)$):** Measures the dispersion or spread around the expected value:
  $$\text{Var}(X) = \mathbb{E}[(X - \mu)^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$$

---

## 2. Common Probability Distributions in Data Science

### Normal (Gaussian) Distribution
The foundational continuous distribution characterized by mean $\mu$ and variance $\sigma^2$:
$$f(x \mid \mu, \sigma^2) = \frac{1}{\sigma \sqrt{2\pi}} \exp\left( -\frac{(x - \mu)^2}{2\sigma^2} \right)$$

### Binomial Distribution
Models the number of successes in $n$ independent Bernoulli trials with success probability $p$:
$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

### Python Implementation
```python
import numpy as np
from scipy import stats

# 1. Simulating a Normal Distribution (e.g., sensor errors or asset metrics)
mu, sigma = 50.0, 5.0
normal_samples = np.random.normal(loc=mu, scale=sigma, size=1000)

print(f"Empirical Mean: {np.mean(normal_samples):.4f}")
print(f"Empirical Std Dev: {np.std(normal_samples):.4f}")

# 2. Calculating Binomial Probability (e.g., 7 successes in 10 trials with p=0.5)
n_trials, p_success = 10, 0.5
k_successes = 7
binom_prob = stats.binom.pmf(k=k_successes, n=n_trials, p=p_success)
print(f"Binomial PMF P(X = {k_successes}): {binom_prob:.4f}")
```
