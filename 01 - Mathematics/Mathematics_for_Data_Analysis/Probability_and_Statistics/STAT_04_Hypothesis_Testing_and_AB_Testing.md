---
tags:
  - mathematics/statistics
  - pro
  - hypothesis-testing
  - ab-testing
  - obsidian-vault
aliases:
  - Hypothesis Testing and AB Testing
---

# Probability and Statistics: Hypothesis Testing and A/B Testing

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[STAT_03_Maximum_Likelihood_Estimation]], [[sklearn_pipelines_deployment]]

---

## 1. Principles of Hypothesis Testing

Hypothesis testing is a statistical framework used to determine whether there is enough evidence in a sample of data to infer that a certain condition is true for a population.

### Key Concepts
* **Null Hypothesis ($H_0$):** Assumes no effect, no difference, or status quo (e.g., model performance or user conversion rate is identical between Group A and Group B).
* **Alternative Hypothesis ($H_1$ or $H_a$):** Assumes an effect or difference exists.
* **Significance Level ($\alpha$):** The threshold probability of rejecting the null hypothesis when it is actually true (typically set to $\alpha = 0.05$).
* **P-value:** The probability of obtaining test results at least as extreme as the observed results, assuming the null hypothesis is true. If $p < \alpha$, we reject $H_0$.

---

## 2. A/B Testing with Welch's t-test

When comparing the means of two independent groups (such as model efficiency scores or website engagement metrics) where variances might be unequal, **Welch's $t$-test** is the gold standard.

### Mathematical Notation
The $t$-statistic for Welch's test is calculated as:
$$t = \frac{\bar{X}_1 - \bar{X}_2}{\sqrt{\frac{s_1^2}{n_1} + \frac{s_2^2}{n_2}}}$$
Where $\bar{X}_i$, $s_i^2$, and $n_i$ represent the sample mean, sample variance, and sample size of group $i$.

### Python Implementation
```python
import numpy as np
from scipy import stats

# Simulated performance metrics for two groups (e.g., SIPDUS asset efficiency or model variations)
group_a = np.array([88.5, 91.0, 89.2, 92.4, 90.1, 88.9], dtype=np.float32)
group_b = np.array([92.1, 94.3, 93.0, 95.2, 91.8, 93.5], dtype=np.float32)

# Perform Welch's t-test (assuming unequal population variances)
t_statistic, p_value = stats.ttest_ind(group_a, group_b, equal_var=False)

print(f"T-statistic: {t_statistic:.4f}")
print(f"P-value: {p_value:.4f}")

alpha = 0.05
if p_value < alpha:
  print("Result: Reject Null Hypothesis (Statistically significant difference detected).")
else:
  print("Result: Fail to Reject Null Hypothesis (No significant difference).")
```
