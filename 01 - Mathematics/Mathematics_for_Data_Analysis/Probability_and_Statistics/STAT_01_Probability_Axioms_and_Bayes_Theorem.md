---
tags:
  - mathematics/probability
  - beginner
  - statistics
  - bayes-theorem
  - obsidian-vault
aliases:
  - Probability Axioms and Bayes Theorem
---

# Probability and Statistics: Axioms, Conditional Probability, and Bayes' Theorem

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[STAT_02_Random_Variables_and_Distributions]], [[STAT_03_Maximum_Likelihood_Estimation]]

---

## 1. Kolmogorov's Probability Axioms

Probability theory provides a mathematical framework for quantifying uncertainty. Let $\Omega$ be the sample space and $E$ be an event. A probability measure $P(E)$ satisfies Kolmogorov's three axioms:
1. **Non-negativity:** For any event $E$, $P(E) \ge 0$.
2. **Unit Measure:** The probability of the entire sample space is 1, i.e., $P(\Omega) = 1$.
3. **Countable Additivity:** For mutually exclusive events $E_1, E_2, \dots$, 
   $$P\left(\bigcup_{i=1}^{\infty} E_i\right) = \sum_{i=1}^{\infty} P(E_i)$$

---

## 2. Conditional Probability and Bayes' Theorem

### Mathematical Notation
* **Conditional Probability:** The probability that event $A$ occurs given that event $B$ has already occurred is defined as:
  $$P(A \mid B) = \frac{P(A \cap B)}{P(B)}, \quad \text{provided } P(B) > 0$$

* **Bayes' Theorem:** By inverting conditional probabilities, Bayes' Theorem relates our prior belief to our posterior belief given new evidence:
  $$P(A \mid B) = \frac{P(B \mid A) P(A)}{P(B)}$$
  Where:
  * $P(A)$ is the **Prior** probability of hypothesis $A$.
  * $P(B \mid A)$ is the **Likelihood** of observing evidence $B$ given $A$.
  * $P(B)$ is the **Evidence** (total marginal probability of $B$).
  * $P(A \mid B)$ is the **Posterior** probability of $A$ given $B$.

### Python Implementation: Bayesian Diagnostic Test
```python
import numpy as np


def bayes_theorem(p_prior: float, p_likelihood: float, p_evidence: float) -> float:
  """Calculates posterior probability P(A|B) using Bayes' Theorem."""
  p_posterior = (p_likelihood * p_prior) / p_evidence
  return p_posterior


# Example: Medical or equipment failure screening
# P(Disease) = 0.01 (1% base rate)
p_disease = 0.01

# P(Positive Test | Disease) = 0.95 (Test Sensitivity)
p_positive_given_disease = 0.95

# P(Positive Test Total) = 0.05 (Total false positives + true positives)
p_positive_total = 0.05

p_disease_given_positive = bayes_theorem(
    p_disease, p_positive_given_disease, p_positive_total
)
print(f"Posterior Probability P(Condition | Positive Test): {p_disease_given_positive:.4f}")
```
