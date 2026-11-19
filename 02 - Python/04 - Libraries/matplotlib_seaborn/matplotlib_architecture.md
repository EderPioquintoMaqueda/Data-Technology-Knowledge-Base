---
id: matplotlib-architecture
title: Matplotlib Module 1 - Object-Oriented Architecture, Figures, and Axes
tags: [python, matplotlib, data-science, visualization, architecture]
aliases: [Matplotlib Architecture, Figure and Axes, OO Plotting]
---

# Matplotlib Module 1: Object-Oriented Architecture, Figures & Axes

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While Pandas and NumPy handle data computation, effective exploratory data analysis and communication require visualization. **Matplotlib** is the foundational plotting library in Python, designed around a hierarchical object-oriented architecture.

---

## 1. Beginner Level: The MATLAB-style vs. Object-Oriented Interface

Matplotlib supports two plotting styles. The MATLAB-style (`pyplot`) relies on implicit state management, which becomes difficult to maintain in complex multi-plot figures. Professional workflows use the **Object-Oriented (OO) Interface** explicitly.

```python
import matplotlib.pyplot as plt
import numpy as np

# Data generation
x = np.linspace(0, 10, 100)
y = np.sin(x)

# Object-Oriented Approach (Recommended)
# fig represents the entire canvas window, ax represents the individual subplot
fig, ax = plt.subplots(figsize=(8, 4))

ax.plot(x, y, color='blue', linewidth=2, label='Sine Wave')
ax.set_title('Basic Object-Oriented Plot')
ax.set_xlabel('X Axis (Radians)')
ax.set_ylabel('Y Amplitude')
ax.legend()
ax.grid(True, linestyle='--', alpha=0.6)

plt.show()
```
