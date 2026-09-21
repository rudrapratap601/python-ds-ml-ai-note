# Seaborn: Statistical Visualization

> **Purpose:** Explore relationships and distributions with concise, data-aware plotting while understanding the aggregation and uncertainty behind each chart. Examples use Seaborn 0.13-style APIs and run in order.

## Contents

- [Setup and tidy data](#setup-and-tidy-data)
- [Axes-level and figure-level functions](#axes-level-and-figure-level-functions)
- [Relationships and semantic mappings](#relationships-and-semantic-mappings)
- [Distributions](#distributions)
- [Categorical comparisons](#categorical-comparisons)
- [Estimates and error bars](#estimates-and-error-bars)
- [Regression, matrices, and multivariate views](#regression-matrices-and-multivariate-views)
- [Faceting and styling](#faceting-and-styling)
- [The objects interface](#the-objects-interface)
- [Pitfalls, export, and practice](#pitfalls-export-and-practice)

## Setup and tidy data

```bash
python -m pip install seaborn
```

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

rng = np.random.default_rng(42)
data = pd.DataFrame({
    "hours": np.tile(np.arange(1, 7), 4),
    "group": np.repeat(["A", "B"], 12),
    "cohort": np.tile(np.repeat(["first", "second"], 6), 2),
})
data["score"] = 40 + 5 * data["hours"] + rng.normal(0, 4, len(data))
sns.set_theme(style="whitegrid", context="notebook")
```

Seaborn builds statistical graphics on Matplotlib. Long/tidy data usually has one observation per row and one variable per column, allowing clear mappings from column names to visual properties.

All data here is synthetic. Avoid relying on `sns.load_dataset` for offline examples because its first use may require a download. Use your own validated table for real work.

## Axes-level and figure-level functions

| Family | Axes-level | Figure-level |
|---|---|---|
| Relationships | `scatterplot`, `lineplot` | `relplot` |
| Distributions | `histplot`, `kdeplot`, `ecdfplot`, `rugplot` | `displot` |
| Categories | `stripplot`, `swarmplot`, `boxplot`, `violinplot`, `barplot`, `pointplot`, `countplot` | `catplot` |
| Regression | `regplot`, `residplot` | `lmplot` |

Axes-level functions accept `ax=` and fit into an existing Matplotlib layout. Figure-level functions create and manage their own figure, often returning a grid object for facets. Do not pass an existing `ax` to a figure-level function expecting it to behave like an axes-level one. See the [function overview](https://seaborn.pydata.org/tutorial/function_overview.html).

```python
fig, ax = plt.subplots(figsize=(6, 4), layout="constrained")
sns.scatterplot(data=data, x="hours", y="score", hue="group", ax=ax)
ax.set_title("Score versus study hours")
```

## Relationships and semantic mappings

`hue` maps color, `style` maps marker/line style, and `size` maps size. Choose mappings that reflect whether a variable is categorical or numeric.

```python
grid = sns.relplot(
    data=data, x="hours", y="score", hue="group", style="group",
    col="cohort", kind="scatter", height=3, aspect=1.1,
)
grid.set_axis_labels("Study hours", "Score")
```

Avoid encoding too many variables at once. Use redundant color/style for an important group, and facet when additional categories would make one panel unreadable.

`lineplot` normally aggregates repeated y values at the same x with an estimator and uncertainty interval. If every row is already one distinct ordered point and you do not want aggregation, specify `estimator=None` and `errorbar=None`, along with grouping such as `units` when needed.

## Distributions

```python
fig, axes = plt.subplots(1, 3, figsize=(11, 3.5), layout="constrained")
sns.histplot(data=data, x="score", bins=8, ax=axes[0])
sns.ecdfplot(data=data, x="score", hue="group", ax=axes[1])
sns.kdeplot(data=data, x="score", hue="group", common_norm=False, ax=axes[2])
```

- **Histogram:** Shows binned counts/density; bin width changes the story.
- **ECDF:** Shows the fraction at or below each value without choosing bins.
- **KDE:** Smooth density estimate; bandwidth and boundary behavior matter.
- **Rug:** Marks individual observations; useful as context for density.

KDE can extend beyond physically possible bounds and is often unsuitable for small, discrete, or highly bounded samples. `cut`, `clip`, and bandwidth settings affect display but do not solve every modeling issue. `common_norm` controls whether group densities share a normalization; unequal sample sizes can change interpretation.

## Categorical comparisons

```python
fig, ax = plt.subplots(figsize=(6, 4), layout="constrained")
sns.boxplot(data=data, x="group", y="score", color="#cbd5e1", ax=ax)
sns.stripplot(data=data, x="group", y="score", color="#1e3a8a", alpha=0.6, jitter=0.15, ax=ax)
ax.set_title("Group distributions with individual observations")
```

| Plot | Encodes | Check |
|---|---|---|
| Strip | Individual observations with optional jitter | Overplotting |
| Swarm | Individual points arranged to reduce overlap | Can be slow/crowded |
| Box | Quartiles, median, whiskers, outlier convention | Not a confidence interval |
| Violin | Estimated density | Bandwidth and sample size |
| Count | Number of observations per category | Missing categories and denominator |
| Bar | Estimated numeric value per category | Default aggregation and uncertainty |
| Point | Estimated values with connected category positions | Whether connecting categories is meaningful |

Specify category order for ordinal variables rather than relying on incidental row order. In modern Seaborn, use `hue` with a palette when colors encode categories; avoid deprecated palette-without-hue patterns.

## Estimates and error bars

```python
fig, ax = plt.subplots(figsize=(6, 4), layout="constrained")
sns.barplot(
    data=data, x="group", y="score", estimator="mean",
    errorbar=("ci", 95), n_boot=500, seed=42,
    hue="group", palette="colorblind", legend=False, ax=ax,
)
ax.set_title("Mean score with bootstrap confidence intervals")
```

Modern functions use `errorbar=`; older examples may use deprecated `ci=` parameters. Choices include confidence intervals, percentile intervals, standard error, standard deviation, or no bars. The [error-bar tutorial](https://seaborn.pydata.org/tutorial/error_bars.html) distinguishes uncertainty in an estimate from spread in the observations.

Standard deviation describes data spread; standard error and confidence intervals describe an estimator under assumptions. Percentile intervals describe a portion of the observed distribution. Label the meaning instead of calling everything an “error range.”

Bootstrap intervals still depend on an appropriate resampling unit and sampling design. Repeated measurements from the same subject are not independent rows. Aggregated bars can hide multimodal or unequal-sized groups; show raw data or distributions when possible.

## Regression, matrices, and multivariate views

```python
fig, axes = plt.subplots(1, 2, figsize=(9, 3.5), layout="constrained")
sns.regplot(data=data, x="hours", y="score", ci=None, ax=axes[0])
sns.residplot(data=data, x="hours", y="score", ax=axes[1])
```

Regression plots are exploratory summaries, not proof of a causal relationship or a substitute for model diagnostics. Some specialized regression options require optional statistical dependencies. Inspect residual structure, outliers, and whether the fitted relationship is appropriate.

```python
correlation = data[["hours", "score"]].corr()
fig, ax = plt.subplots(figsize=(4, 3.5), layout="constrained")
sns.heatmap(correlation, vmin=-1, vmax=1, center=0, cmap="vlag", annot=True, fmt=".2f", square=True, ax=ax)
```

Use fixed comparable bounds for correlation heatmaps. Correlation is sensitive to data selection, outliers, nonlinear structure, and missing-data handling. A large matrix of pairwise correlations involves many comparisons.

`pairplot` creates pairwise relationships and diagonal distributions; choose a small set of meaningful columns. `jointplot` combines a bivariate view with marginal distributions. `clustermap` adds hierarchical clustering; scale features appropriately and understand the distance/linkage choices.

## Faceting and styling

Facet by `row`, `col`, or `col_wrap` to compare groups in separate panels. Use shared scales when direct comparisons require them. Define `hue_order`, category order, and palette mappings consistently across facets.

Figure-level sizing uses `height` per facet and `aspect` as width/height. Changing a global Matplotlib figure size does not control a figure that Seaborn has already created.

Use `sns.set_context` for notebook/talk/poster text scaling, `sns.set_style` for background/axes appearance, and `sns.despine` where appropriate. Apply local styling contexts when writing a library function so it does not alter every later chart.

## The objects interface

Seaborn also offers a compositional `seaborn.objects` interface using marks, mappings, statistics, and moves. It is an alternative API, not a requirement for using the classic functions.

```python
import seaborn.objects as so

plot = so.Plot(data, x="hours", y="score", color="group").add(so.Dot())
rendered = plot.plot()
```

Learn one interface well before mixing both styles. Check its current documentation for supported transformations and behaviors.

## Pitfalls, export, and practice

Axes-level output can be saved through its Matplotlib figure. Figure-level grids expose `savefig`:

```python
from io import BytesIO

buffer = BytesIO()
grid.savefig(buffer, format="png", dpi=120)
assert buffer.getvalue().startswith(b"\x89PNG")
plt.close("all")
```

Common mistakes: confusing counts with means, assuming error bars are standard deviations, treating KDE as observed truth, choosing many nearly identical colors, drawing unaggregated time series with unintended aggregation, and forgetting to state missing-value handling.

Practice: compare histogram and ECDF views; add raw observations to a box plot; create comparable facets with fixed ordering; show how different error-bar choices change interpretation; write an axes-level plotting helper that accepts `ax`.

References: [Seaborn tutorials](https://seaborn.pydata.org/tutorial.html), [API](https://seaborn.pydata.org/api.html). Related: [Matplotlib](matplotlib.md), [SciPy](scipy.md).
