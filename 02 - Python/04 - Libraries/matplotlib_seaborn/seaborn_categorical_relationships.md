---
id: seaborn-categorical-relationships
title: Seaborn Module 4 - Categorical Relationships, Boxplots, and Violin Plots
tags: [python, seaborn, matplotlib, data-science, categorical-data, boxplot]
aliases: [Seaborn Categorical Plots, Boxplots and Violins, Statistical Comparisons]
---

# Seaborn Module 4: Categorical Relationships, Boxplots & Violin Plots

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


When analyzing how a numerical variable changes across distinct categorical groups (e.g., salaries across job titles, project completion times across departments), standard scatter plots are insufficient. Seaborn provides specialized categorical plotting functions (`boxplot`, `violinplot`, `barplot`, `stripplot`) that automatically compute statistical summaries (medians, quartiles, and outliers).

---

## 1. Beginner Level: Boxplots and Swarmplots

A **Box Plot** displays the distribution of quantitative data through its five-number summary: minimum, first quartile (Q1), median, third quartile (Q3), and maximum, along with explicit outlier points.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

sns.set_theme(style='whitegrid')

# Sample dataset
np.random.seed(42)
data = pd.DataFrame({
    'Department': np.repeat(['Engineering', 'Finance', 'Operations', 'Legal'], 100),
    'Salary_k': np.concatenate([
        np.random.normal(65, 10, 100),
        np.random.normal(55, 8, 100),
        np.random.normal(45, 6, 100),
        np.random.normal(60, 12, 100)
    ])
})

# 1. Boxplot
fig, ax = plt.subplots(figsize=(8, 4))
sns.boxplot(data=data, x='Department', y='Salary_k', palette='Set2', ax=ax)

ax.set_title('Salary Distribution by Department (Boxplot)')
ax.set_xlabel('Department')
ax.set_ylabel('Salary ($k)')
plt.show()
```
