# Python GUI Development and Streamlit

> **Purpose:** Understand desktop and browser-based Python interfaces, then build, test, and deploy a useful Streamlit data app. This is a learning reference; library APIs evolve, so record your installed versions and consult the linked official documentation.

## Contents

- [Choosing an interface](#choosing-an-interface)
- [GUI foundations](#gui-foundations)
- [Desktop basics with Tkinter](#desktop-basics-with-tkinter)
- [Streamlit setup and execution](#streamlit-setup-and-execution)
- [Widgets and layout](#widgets-and-layout)
- [State, callbacks, and forms](#state-callbacks-and-forms)
- [Caching and resource lifetime](#caching-and-resource-lifetime)
- [Complete CSV explorer app](#complete-csv-explorer-app)
- [Navigation and project structure](#navigation-and-project-structure)
- [Charts, downloads, and advanced interaction](#charts-downloads-and-advanced-interaction)
- [Testing and deployment](#testing-and-deployment)
- [Other frameworks and practice](#other-frameworks-and-practice)

## Choosing an interface

| Tool | Interface style | Typical use |
|---|---|---|
| Tkinter / ttk | Desktop widgets and event loop | Small local utilities and learning GUI concepts |
| PySide6 / Qt | Rich desktop widgets or QML | Complex desktop applications |
| Streamlit | Python script rendered as a web app | Data exploration, dashboards, model demos |
| Dash | Web layout with explicit callbacks | Analytical applications with coordinated interactions |
| Gradio | Web components around Python functions | ML demonstrations and interactive model interfaces |
| Web framework plus frontend | Explicit server and client architecture | Products needing extensive custom workflows |

Streamlit runs a Python server and displays its interface in a browser. It is not a native desktop widget toolkit. A notebook is another interactive environment, but its execution and deployment model differs from a standalone application.

The official [Qt for Python](https://doc.qt.io/qtforpython-6/) and [Dash tutorial](https://dash.plotly.com/tutorial) describe their respective desktop and callback-based web models.

## GUI foundations

An interface combines widgets, layout, state, events, and application logic. A click or input change triggers work; the interface then reflects updated state.

Keep business logic in ordinary functions that can run without the UI. Validate inputs at the boundary. Represent loading, empty, success, and error states explicitly. Labels should explain units and expected input; do not rely on color alone to convey meaning.

Desktop event loops must remain responsive. Long computations in a callback freeze the interface. Move appropriate work to a worker and marshal results back to the UI thread using the framework's supported mechanism. Do not update desktop widgets directly from arbitrary worker threads.

## Desktop basics with Tkinter

Tkinter is Python's standard interface to Tcl/Tk, though a particular Python installation may need Tk support installed separately. `python -m tkinter` is a useful installation check. See the [Tkinter documentation](https://docs.python.org/3/library/tkinter.html).

Save this standalone example as `desktop_app.py` and run it in a graphical desktop session. It starts an event loop and is not a notebook calculation.

<!-- verify: manual-gui -->
```python
import tkinter as tk
from tkinter import ttk


def main():
    root = tk.Tk()
    root.title("Study timer setup")
    root.columnconfigure(0, weight=1)
    frame = ttk.Frame(root, padding=16)
    frame.grid(sticky="nsew")
    frame.columnconfigure(1, weight=1)
    minutes = tk.StringVar(value="25")
    message = tk.StringVar(value="Choose a positive number of minutes.")

    def apply():
        try:
            value = int(minutes.get())
            if value <= 0:
                raise ValueError
        except ValueError:
            message.set("Enter a positive whole number.")
        else:
            message.set(f"Session configured for {value} minutes.")

    ttk.Label(frame, text="Minutes").grid(row=0, column=0, padx=4)
    ttk.Entry(frame, textvariable=minutes).grid(row=0, column=1, sticky="ew")
    ttk.Button(frame, text="Apply", command=apply).grid(row=1, column=0, columnspan=2)
    ttk.Label(frame, textvariable=message).grid(row=2, column=0, columnspan=2)
    root.mainloop()


if __name__ == "__main__":
    main()
```

Pass a callback such as `command=apply`, not its result `command=apply()`. Use one layout manager consistently within a given parent: avoid mixing `pack` and `grid` among children of the same container. Use `after(milliseconds, callback)` for scheduled UI work instead of blocking `sleep`.

Further desktop topics include menus, dialogs, keyboard bindings, validation, model/view separation, accessibility, worker coordination, application icons, packaging, and platform-specific testing. Packaging a GUI does not remove its native library dependencies.

## Streamlit setup and execution

Use a project environment, then install the libraries the app actually needs:

```bash
python -m venv .venv
# Activate the environment using your shell's normal command.
python -m pip install streamlit pandas plotly
python -m streamlit run app.py
```

Streamlit normally reruns the script from top to bottom when a widget changes. Ordinary local variables are recreated; session state and caching solve different persistence needs. Forms and fragments can change the rerun boundary. See [Streamlit's execution concepts](https://docs.streamlit.io/develop/concepts).

Minimal standalone `app.py`:

<!-- verify: streamlit-minimal -->
```python
import streamlit as st

st.set_page_config(page_title="Study dashboard", layout="wide")
st.title("Study dashboard")
hours = st.slider("Study hours this week", min_value=0, max_value=40, value=5)
st.metric("Planned minutes", hours * 60)
```

Run apps through `streamlit run`, not simply `python app.py`, so they receive the proper application runtime.

## Widgets and layout

| Need | Streamlit tools |
|---|---|
| Text and explanation | `st.title`, `st.header`, `st.markdown`, `st.caption` |
| Text input | `st.text_input`, `st.text_area` |
| Numeric input | `st.number_input`, `st.slider` |
| Choices | `st.selectbox`, `st.multiselect`, `st.radio`, `st.checkbox` |
| Actions | `st.button`, `st.form_submit_button` |
| Dates | `st.date_input`, `st.time_input` |
| Data | `st.dataframe`, `st.data_editor`, `st.table` |
| Layout | `st.sidebar`, `st.columns`, `st.tabs`, `st.expander`, `st.container` |
| Feedback | `st.info`, `st.warning`, `st.error`, `st.success`, `st.spinner` |
| Media | `st.image`, `st.audio`, `st.video` |
| Files | `st.file_uploader`, `st.download_button` |

Give repeated or stateful widgets stable, unique `key` values. A widget returns its current value. A button's `True` value corresponds to the run caused by its click; it is not permanent application state.

Treat hidden layouts as presentation, not a guarantee that their bodies are never computed. Place expensive work behind explicit conditions or suitable lazy execution patterns.

## State, callbacks, and forms

Session state holds values across reruns within a user's session. It is not a database, durable storage, or an authentication boundary. A reconnect or reload can create a new session. See the [session state guide](https://docs.streamlit.io/develop/concepts/architecture/session-state).

Standalone counter app:

<!-- verify: streamlit-counter -->
```python
import streamlit as st

if "count" not in st.session_state:
    st.session_state.count = 0


def increment():
    st.session_state.count += 1


st.button("Add one", on_click=increment, key="increment")
st.metric("Count", st.session_state.count)
```

A callback runs before the subsequent script rerun. Read current keyed widget values from session state when needed. Do not mutate a widget's session-state value after creating that widget in the same run; initialize or update it at the appropriate earlier point.

A form batches edits until submission. Use `with st.form("filters"):` and put `st.form_submit_button("Apply")` inside. Regular widget callbacks are restricted inside forms; the submit button supports a callback. Save submitted results in session state when they should persist during later reruns.

## Caching and resource lifetime

| Mechanism | Store | Sharing/lifetime concern |
|---|---|---|
| `st.session_state` | User interaction state | Session-scoped, not durable |
| `@st.cache_data` | Serializable computation results | Cached results are copied on return; cache can be shared |
| `@st.cache_resource` | Models, clients, other resources | Default shared object may be used across sessions |
| Database / object storage | Durable records | Schema, authorization, transactions, retention |

Set appropriate cache limits such as `ttl` and `max_entries`. Include every result-affecting input in the cache key. Shared cached resources must be safe for concurrent use; mutable per-user state belongs elsewhere. Cache invalidation and identity are explained in the [caching guide](https://docs.streamlit.io/develop/concepts/architecture/caching).

Do not place a write operation inside a cached function if every call is expected to perform the write. Do not exclude an argument from hashing unless it truly cannot change the computed result. Authorization must not be bypassed by a shared cached result.

## Complete CSV explorer app

Save the following complete block as `app.py`. It has a built-in dataset, so it works without an upload. The sample accepts `date`, `category`, and `amount` columns and validates them before plotting.

<!-- verify: streamlit-explorer -->
```python
from io import BytesIO

import pandas as pd
import plotly.express as px
import streamlit as st

st.set_page_config(page_title="CSV explorer", layout="wide")
st.title("CSV explorer")
st.caption("Explore dated amounts by category. Upload UTF-8 CSV data or use the sample.")


def parse_sales(payload):
    frame = pd.read_csv(BytesIO(payload))
    required = {"date", "category", "amount"}
    missing = required - set(frame.columns)
    if missing:
        raise ValueError(f"Missing columns: {', '.join(sorted(missing))}")
    frame = frame.loc[:, ["date", "category", "amount"]].copy()
    frame["date"] = pd.to_datetime(frame["date"], errors="coerce", utc=True)
    frame["amount"] = pd.to_numeric(frame["amount"], errors="coerce")
    frame["category"] = frame["category"].astype("string").str.strip()
    invalid = (
        frame["date"].isna()
        | frame["amount"].isna()
        | frame["amount"].isin([float("inf"), float("-inf")])
        | frame["category"].isna()
        | frame["category"].eq("")
    )
    if invalid.any():
        raise ValueError(f"{int(invalid.sum())} rows contain invalid or missing values.")
    return frame.sort_values("date")


sample = b"date,category,amount\n2026-01-01,Books,120\n2026-01-02,Tools,80\n2026-01-03,Books,60\n"
upload = st.sidebar.file_uploader("Sales CSV", type=["csv"])
if upload is not None and upload.size > 5 * 1024 * 1024:
    st.error("Use a CSV smaller than 5 MiB for this demo.")
    st.stop()

try:
    frame = parse_sales(sample if upload is None else upload.getvalue())
except (ValueError, UnicodeDecodeError, pd.errors.ParserError, pd.errors.EmptyDataError) as exc:
    st.error(f"Unable to read this dataset: {exc}")
    st.stop()

if frame.empty:
    st.info("The dataset has headers but no rows.")
    st.stop()

categories = sorted(frame["category"].unique().tolist())
selected = st.sidebar.multiselect("Categories", categories, default=categories)
filtered = frame.loc[frame["category"].isin(selected)]
if filtered.empty:
    st.info("Select at least one category containing data.")
    st.stop()

left, right = st.columns(2)
left.metric("Rows", len(filtered))
right.metric("Total amount", f"{filtered['amount'].sum():,.2f}")
summary = filtered.groupby("category", as_index=False)["amount"].sum()
st.plotly_chart(px.bar(summary, x="category", y="amount", title="Amount by category"))
st.dataframe(filtered)
st.download_button(
    "Download filtered CSV",
    data=filtered.to_csv(index=False).encode("utf-8"),
    file_name="filtered-sales.csv",
    mime="text/csv",
)
```

This demo reparses a small upload on rerun. For repeated expensive trusted computations, add a deliberate caching policy. A file-extension filter is a usability feature, not complete validation; the parser and application rules still matter. Numeric floats are fine for this teaching dashboard; use an appropriate exact representation when exact monetary accounting is required.

## Navigation and project structure

```text
dashboard/
├── app.py                 # Navigation and shared layout
├── views/
│   ├── overview.py
│   └── quality.py
├── services/
│   └── data.py            # Parsing, validation, transformations
├── tests/
│   └── test_data.py
├── .streamlit/
│   └── config.toml
└── requirements.txt
```

`st.Page` and `st.navigation` provide explicit multipage navigation. The selected page must be run, for example `page = st.navigation([...]); page.run()`. An older directory-based `pages/` convention also exists; choose one approach deliberately. The [navigation API](https://docs.streamlit.io/develop/api-reference/navigation/st.navigation) documents supported page definitions and behavior.

When widgets disappear between pages or conditional runs, their widget-associated state can be cleaned up. Persist durable application selections in separate session-state keys if needed. A hidden navigation entry is not access control.

## Charts, downloads, and advanced interaction

- Use `st.plotly_chart(fig)` for interactive Plotly figures and `st.pyplot(fig)` for explicit Matplotlib figures.
- Use `st.data_editor` for editable data; validate its returned data before persistence.
- `st.stop()` ends the current run. `st.rerun()` requests another run; avoid accidental rerun loops.
- Fragments can rerun a portion of an app; dialogs provide focused interaction. Learn their state and rerun rules before mixing them with full-app control flow.
- Query parameters can encode shareable non-sensitive view settings; validate them like other external input.
- Chat interfaces use `st.chat_message` and `st.chat_input`; save history deliberately and stream responses when useful.
- Custom components extend the UI but add frontend dependencies and maintenance.
- Background jobs need explicit task state and result storage; a long synchronous call still blocks that session's interaction.

For database work, use parameterized queries, scoped credentials, connection management, and explicit authorization. Keep secrets in supported secrets management such as `st.secrets`, never in committed source or browser-visible text.

## Testing and deployment

Test data transformations as ordinary Python functions. Streamlit's `AppTest` can execute apps and simulate supported widget interactions without a browser; it does not replace all browser, layout, or component tests. See [app testing](https://docs.streamlit.io/develop/api-reference/app-testing).

The following assumes the counter app above was saved as `counter.py`:

<!-- verify: external-file -->
```python
from streamlit.testing.v1 import AppTest

app = AppTest.from_file("counter.py").run()
assert not app.exception
app.button[0].click().run()
assert app.session_state["count"] == 1
```

Before deploying, pin and test dependencies, verify startup commands, configure secrets, and test empty/invalid/large inputs. Check host-specific limits and storage behavior. Local disk may be ephemeral on a hosted platform; persistent data belongs in a supported durable service.

Authentication identifies a user; authorization determines what that user may read or change. Test multiple sessions to catch accidental shared state. Monitor errors and slow requests without logging private data unnecessarily. Deployment and authentication details vary by hosting platform; follow the [Streamlit deployment documentation](https://docs.streamlit.io/deploy).

## Other frameworks and practice

**PySide6:** Learn `QApplication`, widgets, layouts, signals and slots, model/view tables, worker coordination, and packaging. Design state independently of widgets so logic remains testable.

**Dash:** Layout components have IDs; callbacks map input properties to output properties. This gives explicit interaction dependencies. Separate server-side data access from browser-visible component data.

**Gradio:** Start with function inputs and outputs, then use its layout/event APIs for richer model workflows. Review its current [official guides](https://www.gradio.app/guides) for supported components and deployment behavior.

Practice projects: add date filtering to the explorer; build a validation report instead of rejecting the entire file; persist saved reports in SQLite; create a multipage app with separate exploration and data-quality pages; reproduce a small desktop form with Tkinter.

**Remember:** Understand the framework's execution model first. Most confusing GUI bugs involve state, callback timing, resource lifetime, or work running in the wrong execution context.
