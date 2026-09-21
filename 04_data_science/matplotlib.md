# Matplotlib: Precise Statistical and Scientific Figures

> **Purpose:** Build clear figures with explicit control over axes, annotations, scales, layouts, and export. Examples run in order; they construct figures without requiring a downloaded dataset. Call `plt.show()` when you want an interactive display.

## Contents

- [Setup and object model](#setup-and-object-model)
- [Choosing chart types](#choosing-chart-types)
- [Labels, styles, and legends](#labels-styles-and-legends)
- [Subplots and layout](#subplots-and-layout)
- [Scales, ticks, and dates](#scales-ticks-and-dates)
- [Images, heatmaps, and color](#images-heatmaps-and-color)
- [Uncertainty and annotations](#uncertainty-and-annotations)
- [Export and resource management](#export-and-resource-management)
- [Advanced topics and performance](#advanced-topics-and-performance)
- [Visual checks and practice](#visual-checks-and-practice)

## Setup and object model

```bash
python -m pip install matplotlib
```

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.array([1, 2, 3, 4])
y = np.array([2, 3, 5, 4])
fig, ax = plt.subplots(figsize=(6, 4), layout="constrained")
ax.plot(x, y, marker="o", label="Observed")
ax.set(title="Study sessions", xlabel="Week", ylabel="Hours")
ax.legend()
assert len(ax.lines) == 1
```

- A **Figure** is the whole canvas.
- An **Axes** is a plotting area containing data, labels, and usually two axes.
- An **Axis** manages one coordinate dimension, including ticks and labels.
- **Artists** are drawable objects: lines, text, patches, legends, and more.

Prefer the explicit `fig, ax` object-oriented interface for reusable plotting functions and multi-panel figures. `pyplot` manages figure creation/display and also offers an implicit stateful interface. See the [Matplotlib quick start](https://matplotlib.org/stable/users/explain/quick_start.html).

## Choosing chart types

| Question | Chart | Main method |
|---|---|---|
| Change over ordered x values | Line | `ax.plot` |
| Relationship between two quantities | Scatter | `ax.scatter` |
| Compare categories | Bar / horizontal bar | `ax.bar`, `ax.barh` |
| One numeric distribution | Histogram / ECDF | `ax.hist`, or construct sorted cumulative values |
| Compare distributions | Box / violin | `ax.boxplot`, `ax.violinplot` |
| Matrix or image | Heatmap / image | `ax.imshow`, `ax.pcolormesh` |
| Estimate with uncertainty | Error bars / band | `ax.errorbar`, `ax.fill_between` |
| Dense point cloud | Hexbin | `ax.hexbin` |

Sort x values before a line plot when the intended ordering is numerical or chronological. A line connects points in the order supplied. Bars usually need a zero baseline because their lengths encode magnitude.

```python
rng = np.random.default_rng(42)
values = rng.normal(size=200)
fig, axes = plt.subplots(1, 2, figsize=(9, 3.5), layout="constrained")
axes[0].hist(values, bins=20, edgecolor="white")
axes[0].set(title="Distribution", xlabel="Value", ylabel="Count")
axes[1].scatter(values, 2 * values + rng.normal(size=200), alpha=0.5, s=18)
axes[1].set(title="Association", xlabel="Input", ylabel="Response")
```

Bin choice changes a histogram's appearance. Density normalizes area, not the height of every bar to a probability. Use aggregation, alpha, or density views for overplotting instead of drawing millions of opaque markers.

## Labels, styles, and legends

Use descriptive titles, units on axes, readable tick labels, and a legend only when it adds information. A legend does not replace an axis label.

```python
with plt.rc_context({"font.size": 10, "axes.spines.top": False, "axes.spines.right": False}):
    fig, ax = plt.subplots(figsize=(6, 3.5), layout="constrained")
    ax.plot([1, 2, 3], [4, 6, 5], color="#2563eb", linewidth=2, marker="o", label="Baseline")
    ax.plot([1, 2, 3], [3, 5, 7], color="#c2410c", linestyle="--", marker="s", label="Candidate")
    ax.set(xlabel="Epoch", ylabel="Validation loss", title="Loss comparison")
    ax.grid(axis="y", alpha=0.2)
    ax.legend(frameon=False)
```

`rc_context` limits style changes to a block. Global `rcParams` or `plt.style.use` changes can affect unrelated figures. Combine color with line style or markers for accessibility.

Use `label=` when creating artists and then `ax.legend()`. If the legend has duplicate entries, correct the artist labels or choose explicit handles rather than manually editing rendered text.

## Subplots and layout

```python
fig, axes = plt.subplots(2, 2, figsize=(8, 6), sharex=True, layout="constrained")
for number, axis in enumerate(axes.flat, start=1):
    axis.plot([0, 1, 2], [0, number, number * 2])
    axis.set_title(f"Scenario {number}")
fig.supxlabel("Time")
fig.supylabel("Response")
```

The shape of `axes` depends on the subplot arrangement. `squeeze=False` requests a consistent 2D array. `subplot_mosaic` provides named layouts; `GridSpec` provides finer control over relative placement.

Shared axes support comparison only when units and ranges are meaningfully comparable. `layout="constrained"` helps handle labels, legends, and colorbars. Avoid mixing competing layout engines without understanding their interaction.

## Scales, ticks, and dates

```python
from matplotlib.ticker import MaxNLocator, StrMethodFormatter

fig, ax = plt.subplots(figsize=(6, 3.5), layout="constrained")
ax.plot([1, 2, 3, 4], [100, 1000, 10000, 100000])
ax.set_yscale("log")
ax.xaxis.set_major_locator(MaxNLocator(integer=True))
ax.set(xlabel="Step", ylabel="Count (log scale)")
```

Log scales require appropriate positive values; `symlog` can represent signed values with a linear region near zero. Explain transformed scales in the figure. Changing limits can hide outliers or exaggerate differences; choose them deliberately.

Locators choose tick positions; formatters choose displayed text. Prefer them over setting only tick label strings, which can become detached from actual positions.

```python
import datetime as dt
import matplotlib.dates as mdates

dates = [dt.datetime(2026, 1, day) for day in range(1, 5)]
fig, ax = plt.subplots(figsize=(6, 3.5), layout="constrained")
ax.plot(dates, [3, 4, 2, 5], marker="o")
locator = mdates.AutoDateLocator()
ax.xaxis.set_major_locator(locator)
ax.xaxis.set_major_formatter(mdates.ConciseDateFormatter(locator))
ax.set(ylabel="Events", title="Daily events")
```

For dual axes, prefer a secondary axis with an actual unit conversion. Unrelated two-scale overlays can create misleading visual relationships; separate panels are often easier to interpret.

## Images, heatmaps, and color

```python
matrix = np.array([[1, 2, 3], [4, 5, 6]])
fig, ax = plt.subplots(figsize=(5, 3), layout="constrained")
image = ax.imshow(matrix, cmap="viridis", aspect="auto")
fig.colorbar(image, ax=ax, label="Count")
ax.set(xlabel="Feature", ylabel="Group", title="Counts by group")
```

Use sequential palettes for ordered magnitude, diverging palettes for deviations around a meaningful center, and qualitative palettes for categories. A colorbar describes numeric mapping; a legend identifies discrete groups or artist types.

`imshow` treats data as a regular image; `pcolormesh` can express coordinate grids more explicitly. Check image origin, aspect ratio, extent, and interpolation. For comparisons across panels, use the same color normalization when the meaning should be shared.

## Uncertainty and annotations

```python
x = np.array([1, 2, 3, 4])
estimate = np.array([4., 5., 5.5, 6.])
margin = np.array([0.5, 0.4, 0.6, 0.5])
fig, ax = plt.subplots(figsize=(6, 3.5), layout="constrained")
ax.plot(x, estimate, color="#2563eb", label="Estimate")
ax.fill_between(x, estimate - margin, estimate + margin, alpha=0.2, label="Illustrative interval")
ax.annotate("Largest estimate", xy=(4, 6), xytext=(2.4, 7), arrowprops={"arrowstyle": "->"})
ax.set(xlabel="Condition", ylabel="Response", ylim=(3, 7.5))
ax.legend()
```

These margins are illustrative, not calculated confidence intervals. Label whether uncertainty represents standard deviation, standard error, a confidence interval, a prediction interval, or another quantity. Those meanings are different.

Annotation coordinates can use data, axes fractions, or figure fractions. `ax.transAxes` expresses positions relative to the plotting area, useful for stable panel labels. Keep annotations away from important data.

## Export and resource management

```python
from io import BytesIO

output = BytesIO()
fig.savefig(output, format="png", dpi=160, bbox_inches="tight")
assert output.getvalue().startswith(b"\x89PNG")
plt.close(fig)
```

`fig.savefig("chart.svg")` or PDF creates vector output where artists support it. PNG is raster output. Figure size is in inches; pixel dimensions depend on DPI. Export before closing the figure and verify the saved artifact, not only the notebook preview.

For automated batch jobs, select a noninteractive backend such as Agg before importing `pyplot` if the environment requires it. Close figures after use to avoid retaining many canvases. Prefer saving through an explicit `fig` reference over whichever figure happens to be current.

## Advanced topics and performance

Transforms connect data coordinates to screen/display coordinates. Artists can be updated after creation; animations use tools such as `FuncAnimation`. Keep the animation object alive and check writer dependencies when exporting video or GIF.

For large plots, aggregate or downsample based on the question, use collections instead of many individual artists, and consider rasterizing dense layers in vector exports. Reducing points should preserve important extrema or temporal structure.

Three-dimensional axes are available but can hide structure through perspective and occlusion. Small multiples or projections are often clearer. Interactive backends support event callbacks, but the application must manage event-loop and thread behavior.

## Visual checks and practice

Check the final output at intended size: are labels clipped, fonts readable, units correct, legends unambiguous, colors distinguishable, and data ranges honest? Numerical assertions verify data, while image inspection verifies layout; neither replaces the other.

Practice: create a three-panel model evaluation figure; share color scales across heatmaps; show a time series with a justified interval; export both SVG and PNG; compare the effect of histogram bin choices; write a plotting function that accepts an `ax` instead of creating a global figure.

```python
plt.close("all")
```

References: [Matplotlib tutorials](https://matplotlib.org/stable/tutorials/index.html), [API](https://matplotlib.org/stable/api/index.html). Related: [Seaborn](seaborn.md), [Plotly](plotly.md).
