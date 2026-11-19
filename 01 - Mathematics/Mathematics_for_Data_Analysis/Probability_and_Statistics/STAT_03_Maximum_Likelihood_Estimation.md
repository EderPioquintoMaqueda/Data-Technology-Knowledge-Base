---
tags:
  - mathematics/statistics
  - pro
  - mle
  - estimation
  - obsidian-vault
aliases:
  - Maximum Likelihood Estimation
---

# Probability and Statistics: Maximum Likelihood Estimation (MLE)

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[STAT_02_Random_Variables_and_Distributions]], [[STAT_04_Hypothesis_Testing_and_AB_Testing]]

---

## 1. Principles of Maximum Likelihood Estimation

Maximum Likelihood Estimation is a foundational framework for estimating the parameters $\theta$ of a probability distribution given a set of observed data. The core idea is to find the parameter values that maximize the **likelihood function**—making the observed data samples "most probable".

### Mathematical Notation
Given independent and identically distributed (i.i.d.) data samples $x_1, x_2, \dots, x_n$ drawn from a probability density function $f(x \mid \theta)$, the likelihood function $L(\theta)$ is the product of individual probabilities:
$$L(\theta) = \prod_{i=1}^{n} f(x_i \mid \theta)$$

Because products are computationally difficult to maximize (and prone to underflow), we take the natural logarithm to form the **Log-Likelihood Function** $\ell(\theta)$:
$$\ell(\theta) = \ln L(\theta) = \sum_{i=1}^{n} \ln f(x_i \mid \theta)$$

To find the optimal parameter $\hat{\theta}_{\text{MLE}}$, we differentiate the log-likelihood with respect to $\theta$ and set the derivative to zero:
$$\frac{\partial \ell(\theta)}{\partial \theta} = 0$$

---

## 2. Python Implementation: MLE for a Normal Distribution

For a normal distribution with unknown mean $\mu$ and known variance $\sigma^2 = 1$, maximizing the log-likelihood yields the sample mean as the MLE estimator: $\hat{\mu} = \frac{1}{n}\sum_{i=1}^{n} x_i$.

```python
import numpy as np

# Observed data samples
np.random.seed(42)
true_mu = 25.5
data = np.random.normal(loc=true_mu, scale=1.0, size=500)


# Log-likelihood function for a Normal Distribution with sigma=1
def gaussian_log_likelihood(mu_param, observations):
  n = len(observations)
  # log L(mu) = -n/2 * ln(2*pi) - 0.5 * sum((x_i - mu)^2)
  log_likelihood = -0.5 * np.sum((observations - mu_param) ** 2)
  return log_likelihood


# Grid search to find MLE estimate for mu
mu_candidates = np.linspace(24.0, 27.0, 1000)
log_likelihoods = [gaussian_log_likelihood(mu, data) for mu in mu_candidates]

optimal_mu = mu_candidates[np.argmax(log_likelihoods)]
print(f"True Mean: {true_mu}")
print(f"Estimated MLE Mean (mu_hat): {optimal_mu:.4f}")
print(f"Sample Mean (Verification): {np.mean(data):.4f}")
```
