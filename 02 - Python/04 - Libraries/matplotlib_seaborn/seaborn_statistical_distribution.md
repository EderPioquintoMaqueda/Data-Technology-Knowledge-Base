---
id: seaborn-statistical-distributions
title: Seaborn Module 3 - Statistical Distributions and Univariate Analysis
tags: [python, seaborn, matplotlib, data-science, visualization, statistics]
aliases: [Seaborn Distributions, Histograms and KDE, Statistical Plots]
---

# Seaborn Module 3: Statistical Distributions & Univariate Analysis

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While Matplotlib gives you pixel-level control over raw canvas drawing, **Seaborn** is built on top of Matplotlib specifically for statistical graphics. Seaborn automatically handles complex data aggregations, statistical estimations, and aesthetic styling out of the box.

---

## 1. Beginner Level: Histograms and Kernel Density Estimates (`histplot` & `kdeplot`)

When exploring a continuous numerical variable (e.g., employee salaries, project budgets, or sensor telemetry), the first step is understanding its underlying probability distribution.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# Setting Seaborn modern aesthetic theme
sns.set_theme(style='whitegrid')

# Sample data generation
np.random.seed(42)
data = pd.DataFrame({
    'Score': np.random.normal(loc=75, scale=10, size=500)
})

# 1. Histogram with Kernel Density Estimate (KDE) overlay
fig, ax = plt.subplots(figsize=(8, 4))
sns.histplot(data['Score'], kde=True, color='teal', bins=20, ax=ax)

ax.set_title('Distribution of Test Scores with KDE Curve')
ax.set_xlabel('Score')
ax.set_ylabel('Count / Density')
plt.show()
```
