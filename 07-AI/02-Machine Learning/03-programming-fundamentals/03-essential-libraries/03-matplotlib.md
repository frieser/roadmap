---
tags: ['ai', 'roadmap']
---

## Summary
**Matplotlib** is the most widely used library for creating static, animated, and interactive visualizations in Python. In Machine Learning, it serves as the foundation for **Exploratory Data Analysis (EDA)**, allowing engineers to visualize data distributions, identify outliers, and track model training progress through loss and accuracy curves. It provides both a simple MATLAB-like interface (`pyplot`) and a powerful Object-Oriented API for complex customizations.

## Detailed Explanation

### 1. The Two Interfaces
Matplotlib can be used in two distinct ways, and understanding the difference is crucial for effective plotting.

*   **Pyplot Interface (State-based)**:
    - Imported as `import matplotlib.pyplot as plt`.
    - It maintains an internal "state" of the current figure and axes.
    - Simple and quick for interactive work, but can become confusing in complex scripts.
*   **Object-Oriented (OO) Interface**:
    - Explicitly creates `Figure` and `Axes` objects.
    - Offers better control and is recommended for production code, subplots, and reusable functions.

### 2. Anatomy of a Figure
To master Matplotlib, you must understand its hierarchy:
*   **Figure**: The entire window or page. It tracks all the child Axes and special "artists" (titles, legends, etc.).
*   **Axes**: The "plot" itself. It is the region where data is plotted. A Figure can have many Axes.
*   **Axis**: These are the number-line-like objects (X and Y) that take care of generating graph limits, ticks, and tick labels.

### 3. Basic Implementation (Python)

#### A. Standard Plotting (OO Approach)
```python
import matplotlib.pyplot as plt
import numpy as np

# Generate data
x = np.linspace(0, 10, 100)
y = np.sin(x)

# 1. Create Figure and Axes
fig, ax = plt.subplots(figsize=(8, 4))

# 2. Plot data
ax.plot(x, y, label='Sine Wave', color='#1f77b4', linewidth=2, linestyle='--')

# 3. Customization (Decoration)
ax.set_title('Simple Sine Wave', fontsize=14, fontweight='bold')
ax.set_xlabel('Time (s)')
ax.set_ylabel('Amplitude')
ax.grid(True, linestyle=':', alpha=0.6)
ax.legend()

# 4. Display or Save
plt.tight_layout()
# plt.savefig('sine_wave.png', dpi=300)
plt.show()
```

#### B. Working with Subplots
```python
# Create a 1x2 grid of plots
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

# First plot: Scatter
ax1.scatter(np.random.rand(50), np.random.rand(50), color='purple', alpha=0.5)
ax1.set_title('Random Scatter')

# Second plot: Histogram
ax2.hist(np.random.randn(1000), bins=30, color='orange', edgecolor='black')
ax2.set_title('Normal Distribution')

plt.show()
```

### 4. Customization & Styling
*   **Global Styles**: Use `plt.style.use('ggplot')` or `plt.style.use('seaborn-v0_8')` to instantly change the look.
*   **Colormaps**: Useful for heatmaps (e.g., `cmap='viridis'`, `cmap='coolwarm'`).
*   **Annotations**: Use `ax.annotate()` to point out specific features (like a local minimum/maximum in a loss curve).

## Interview Questions

**Q: What is the difference between a "Figure" and an "Axes" object?**
**A:** A **Figure** is the top-level container for all plot elements (the "canvas"). An **Axes** is the actual region where the data is plotted (the "chart"). One Figure can contain multiple Axes (e.g., in a subplot grid), but an Axes can only belong to one Figure.

**Q: Why is the Object-Oriented (OO) interface generally preferred over the Pyplot interface?**
**A:** The OO interface is more explicit and less prone to errors in complex applications. It allows you to keep track of specific Axes objects directly, making it easier to modify them independently or pass them to functions. The Pyplot interface relies on a "current figure/axes" state, which can be easily overwritten in large scripts.

**Q: How do you handle overlapping labels in a grid of subplots?**
**A:** Use `plt.tight_layout()` or `fig.subplots_adjust()`. `tight_layout()` automatically adjusts subplot parameters so that subplots fit into the figure area without overlapping labels or titles.

**Q: How would you visualize a Correlation Matrix using Matplotlib?**
**A:** You can use `ax.imshow()` or `ax.pcolormesh()` to display the matrix as an image/heatmap. It is common to pair this with `fig.colorbar(im)` to show the scale and `ax.set_xticks()` / `ax.set_yticks()` to label the features.

**Q: How do you save a plot with high resolution?**
**A:** Use `fig.savefig('filename.png', dpi=300)`. The `dpi` (dots per inch) parameter controls the resolution. Higher values are better for print or high-quality documentation.
