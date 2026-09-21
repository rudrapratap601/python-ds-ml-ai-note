# Plotly: Interactive Charts and Dashboards

> **Purpose:** Create interactive figures, customize traces and layouts, export HTML, and understand the boundary between a chart and an application. Examples run in order and do not require network datasets.

## Contents

- [Setup and figure model](#setup-and-figure-model)
- [Plotly Express](#plotly-express)
- [Graph objects and customization](#graph-objects-and-customization)
- [Subplots and secondary axes](#subplots-and-secondary-axes)
- [Hover, categories, and color](#hover-categories-and-color)
- [Time series and animation](#time-series-and-animation)
- [Maps, 3D, and specialized charts](#maps-3d-and-specialized-charts)
- [HTML and static export](#html-and-static-export)
- [Dashboards and performance](#dashboards-and-performance)
- [Testing and practice](#testing-and-practice)

## Setup and figure model

```bash
python -m pip install plotly pandas
```

```python
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

sales = pd.DataFrame({
    "month": [1, 2, 3, 1, 2, 3],
    "region": ["North", "North", "North", "South", "South", "South"],
    "revenue": [100, 130, 120, 90, 110, 140],
    "orders": [10, 12, 11, 9, 10, 13],
})
```

A Plotly figure contains **data** (traces), **layout** (axes, titles, annotations, legends), and optionally **frames** for animation. Plotly Express is a high-level constructor; graph objects provide explicit control. Express returns a graph-object `Figure`, so both APIs compose. See [graph objects](https://plotly.com/python/graph-objects/).

`fig.show()` displays through the configured renderer. In notebooks, renderer support depends on the environment. Building or exporting an ordinary chart does not require a Plotly account.

## Plotly Express

```python
fig = px.line(
    sales.sort_values(["region", "month"]),
    x="month", y="revenue", color="region", markers=True,
    labels={"month": "Month", "revenue": "Revenue (units)", "region": "Region"},
    title="Monthly revenue by region",
)
assert len(fig.data) == 2
```

| Question | Express constructor |
|---|---|
| Relationship | `px.scatter` |
| Ordered change | `px.line`, `px.area` |
| Category comparison | `px.bar` |
| Distribution | `px.histogram`, `px.box`, `px.violin`, `px.ecdf` |
| Matrix/image | `px.imshow` |
| Hierarchy | `px.treemap`, `px.sunburst` |
| Multivariate exploration | `px.scatter_matrix`, `px.parallel_coordinates` |
| Geographic values | Map or geo constructors appropriate to the data |

`px.bar` generally draws the rows you give it; aggregate the table first when you want category totals. `px.histogram` performs binning/aggregation according to its settings. Lines connect input order, so sort explicitly.

```python
totals = sales.groupby("region", as_index=False)["revenue"].sum()
bars = px.bar(totals, x="region", y="revenue", color="region", text_auto=True)
bars.update_layout(showlegend=False, title="Total revenue")
assert len(bars.data) == 2
```

Wide-form inputs can plot several value columns; long-form data gives explicit semantic mappings for color, facets, hover, and animation. See the [Express guide](https://plotly.com/python/plotly-express/).

## Graph objects and customization

```python
custom = go.Figure()
custom.add_trace(go.Scatter(x=[1, 2, 3], y=[4, 6, 5], mode="lines+markers", name="Baseline"))
custom.add_trace(go.Scatter(x=[1, 2, 3], y=[3, 5, 7], mode="lines+markers", name="Candidate"))
custom.update_layout(
    title="Model comparison", template="plotly_white",
    xaxis_title="Iteration", yaxis_title="Score", legend_title_text="Model",
)
custom.update_traces(line={"width": 3})
custom.update_xaxes(dtick=1)
assert custom.layout.xaxis.title.text == "Iteration"
```

`update_traces` changes trace properties; `update_layout` changes figure-level structure. `update_xaxes` and `update_yaxes` target axes, including subplot axes. Use trace selectors when only one type or group should change.

Shapes are layout annotations such as reference lines or rectangles. They are not equivalent to data traces for every legend, hover, or event feature. `add_hline`, `add_vline`, and annotation helpers simplify common thresholds.

## Subplots and secondary axes

```python
from plotly.subplots import make_subplots

panels = make_subplots(rows=1, cols=2, subplot_titles=["Revenue", "Orders"])
north = sales.loc[sales["region"] == "North"]
panels.add_trace(go.Scatter(x=north["month"], y=north["revenue"], name="Revenue"), row=1, col=1)
panels.add_trace(go.Bar(x=north["month"], y=north["orders"], name="Orders"), row=1, col=2)
panels.update_xaxes(title_text="Month")
panels.update_yaxes(title_text="Revenue", row=1, col=1)
panels.update_yaxes(title_text="Order count", row=1, col=2)
```

`make_subplots` requires compatible subplot types for Cartesian, domain, 3D, polar, and other traces. Facets in Express are often easier for repeated views of one dataset.

Secondary axes can represent different units, but unrelated scales can imply false relationships. Use separate panels when comparison would otherwise depend on arbitrary axis ranges.

## Hover, categories, and color

```python
scatter = px.scatter(
    sales, x="orders", y="revenue", color="region", size="revenue",
    hover_data={"month": True}, custom_data=["month"],
    category_orders={"region": ["North", "South"]},
    color_discrete_map={"North": "#2563eb", "South": "#c2410c"},
)
scatter.update_traces(
    hovertemplate="Orders: %{x}<br>Revenue: %{y:.1f}<br>Month: %{customdata[0]}<extra></extra>"
)
```

`hovertemplate` controls tooltip formatting; `<extra></extra>` hides the secondary trace-name box in this example. Anything embedded in figure data or `customdata` may be available in the exported HTML/browser even if it is not visibly plotted. Do not put private fields there merely because they are hidden from the default tooltip.

Use discrete color for categories and continuous color scales for quantities. If numeric values are actually codes, convert them to categories deliberately. Fix color mappings and category order across related charts so meanings do not change between views.

Set log axes only for suitable data, provide meaningful axis labels, and state whether a bar view is stacked, grouped, or normalized. Interactive zoom does not compensate for misleading initial scales.

## Time series and animation

```python
timeline = pd.DataFrame({
    "date": pd.date_range("2026-01-01", periods=5, freq="D"),
    "value": [4, 5, 3, 6, 7],
})
time_figure = px.line(timeline, x="date", y="value", markers=True)
time_figure.update_xaxes(rangeslider_visible=True)

animated = px.scatter(
    sales, x="orders", y="revenue", color="region",
    animation_frame="month", animation_group="region",
    range_x=[0, 15], range_y=[0, 160],
)
assert len(animated.frames) == 3
```

Sort time values and use proper datetime types. Range sliders are useful for navigation but can add clutter. `animation_group` identifies an entity across frames. Fix relevant ranges and category mappings across frames to avoid apparent changes caused only by rescaling.

Animations can hide precise comparisons and be less accessible than small multiples. Include a static summary when readers need to compare many times at once.

## Maps, 3D, and specialized charts

Map APIs and tile providers change over time. Prefer the current map constructors documented for your Plotly version and check provider attribution/token requirements. Longitude/latitude order, coordinate reference systems, and geographic join identifiers must match the chosen chart.

Three-dimensional scatter and surface charts can reveal geometry but also introduce occlusion. Use meaningful aspect ratios and supply projections or alternative views. Sankey, funnel, waterfall, financial, polar, and ternary charts each encode specialized relationships; select them because the data has that structure.

A map does not imply that counts should be compared without population or exposure normalization. Geographic area and color can dominate perception, so document the denominator.

## HTML and static export

```python
html = fig.to_html(full_html=True, include_plotlyjs=True)
assert "Plotly.newPlot" in html
assert "Monthly revenue" in html
```

`fig.write_html("chart.html", include_plotlyjs=True)` produces a larger self-contained chart for ordinary offline use. `include_plotlyjs="cdn"` produces a smaller file that needs network access to load the library. External map tiles or other external resources can still require connectivity even when the main JavaScript is embedded.

Static export such as `fig.write_image("chart.png")` requires compatible Kaleido tooling; modern Kaleido also needs a compatible Chrome/Chromium installation. Static images lose hover and zoom. See [static image export](https://plotly.com/python/static-image-export/) for current requirements.

SVG/PDF are not guaranteed to contain only vectors: WebGL-rendered traces can be embedded as raster layers. Choose dimensions, scale, font availability, and background explicitly, then inspect the exported artifact.

## Dashboards and performance

A standalone Plotly figure supports browser-side chart interactions. Executing arbitrary Python in response to a click requires an application runtime or a compatible widget environment.

- **Streamlit:** Embed with `st.plotly_chart(fig)` and use supported event/state features when needed.
- **Dash:** Connect component inputs/outputs with callbacks for coordinated application behavior.
- **FigureWidget:** Integrate with compatible notebook widget environments and dependencies.

See the [Streamlit guide](../02_advanced_python/gui-development-and-streamlit.md) for an entire small app.

For large data, aggregate server-side, reduce hover payloads, avoid huge numbers of traces, and consider WebGL (`Scattergl` or supported rendering modes) for dense points. WebGL has browser/GPU limitations and does not make unlimited data cheap. Measure serialization size and browser responsiveness as well as Python execution time.

Persisting selection or zoom across app updates requires deliberate state handling; settings such as `uirevision` can help in suitable update workflows. Client-visible data remains client-visible even when a trace is initially hidden.

## Testing and practice

Check trace counts, data values, axis labels, category order, and exported HTML. Then inspect the chart in a browser for clipping, interaction, accessibility, and loading behavior. Figure serialization success does not prove that every renderer or export dependency works.

Practice: build a filtered regional dashboard; add hover units and an explicit category palette; create a static and animated comparison; export a self-contained HTML chart; compare SVG and WebGL performance; explain which interactions need a Python server.

References: [figure creation and updates](https://plotly.com/python/creating-and-updating-figures/), [API](https://plotly.com/python-api-reference/).
