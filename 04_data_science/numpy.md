# NumPy: Arrays and Numerical Computing

> **Purpose:** Learn array shapes, indexing, broadcasting, numerical operations, random sampling, linear algebra, and practical performance habits. Examples in this file can be run in order in one Python session.

## Contents

- [Setup and the array model](#setup-and-the-array-model)
- [Creation and dtypes](#creation-and-dtypes)
- [Indexing, masks, copies, and views](#indexing-masks-copies-and-views)
- [Shapes and broadcasting](#shapes-and-broadcasting)
- [Ufuncs, reductions, and missing values](#ufuncs-reductions-and-missing-values)
- [Sorting, combining, and searching](#sorting-combining-and-searching)
- [Linear algebra](#linear-algebra)
- [Random numbers and reproducibility](#random-numbers-and-reproducibility)
- [Files and interoperability](#files-and-interoperability)
- [Practical feature preprocessing](#practical-feature-preprocessing)
- [Advanced topics, pitfalls, and practice](#advanced-topics-pitfalls-and-practice)

## Setup and the array model

```bash
python -m pip install numpy
```

```python
import numpy as np

array = np.array([[2, 4, 6], [1, 3, 5]], dtype=np.float64)
assert array.shape == (2, 3)
assert array.ndim == 2
assert array.size == 6
assert array.itemsize == 8
assert array.nbytes == 48
```

An `ndarray` has a shape, dtype, strides, and a data buffer. Numeric arrays store fixed-width elements; an object array stores references to Python objects and often loses the performance advantages of numeric arrays.

`shape` gives lengths along axes; `ndim` counts axes; `size` counts elements. `nbytes` counts element-buffer bytes, not every associated Python object or referenced object. The [NumPy beginner guide](https://numpy.org/doc/stable/user/absolute_beginners.html) introduces this vocabulary.

NumPy is a numerical array library, not a labeled table library. Use pandas or Polars when heterogeneous named columns and relational operations are central.

## Creation and dtypes

```python
zeros = np.zeros((2, 3))
ones = np.ones((2, 3), dtype=np.int64)
identity = np.eye(3)
steps = np.arange(0, 10, 2)
grid = np.linspace(0, 1, 5)
assert steps.tolist() == [0, 2, 4, 6, 8]
np.testing.assert_allclose(grid, [0, 0.25, 0.5, 0.75, 1])
```

| Constructor | Use |
|---|---|
| `np.array(data, dtype=...)` | Create an array from data |
| `np.asarray(data, dtype=...)` | Convert, reusing storage where compatible |
| `np.zeros`, `ones`, `full` | Initialize to a known value |
| `np.empty` | Allocate without initializing numeric contents; fill before reading |
| `np.arange` | Half-open steps; especially useful for integer ranges |
| `np.linspace` | A specified number of samples, including the end by default |
| `np.eye` | Identity-like matrix |
| `zeros_like`, `full_like` | Match another array's shape and usually dtype |

Floating-point steps in `arange` can produce surprising endpoints; use `linspace` when the sample count and endpoints are the contract.

Common dtypes include `bool`, signed/unsigned integers, floating point, complex values, `datetime64`, and `timedelta64`. Fixed-width integer arithmetic can overflow; it does not behave like Python's arbitrary-precision `int`. Casting a float to an integer truncates fractional information.

```python
values = np.array([1, 2, 3], dtype=np.int32)
floating = values.astype(np.float64)
assert floating.dtype == np.float64
assert np.issubdtype(values.dtype, np.integer)
```

`astype` commonly allocates; `copy=False` permits reuse only when possible. Do not rely on implicit dtype promotion across library versions without tests, especially when mixing Python scalars with low-precision arrays.

## Indexing, masks, copies, and views

```python
matrix = np.arange(12).reshape(3, 4)
assert matrix[1, 2] == 6
assert matrix[-1, -1] == 11
assert matrix[:, 1].tolist() == [1, 5, 9]
assert matrix[1:, :2].tolist() == [[4, 5], [8, 9]]

view = matrix[:, :2]
assert np.shares_memory(matrix, view)
selected = matrix[[0, 2]]
assert not np.shares_memory(matrix, selected)
```

Basic slicing generally produces a view sharing data. Advanced integer-array and boolean indexing produce copies. Assignment through an indexed expression still targets the original array; extracting a copied array and then changing that copy is different.

```python
values = np.array([1, 4, 7, 10])
mask = (values >= 4) & (values < 10)
assert values[mask].tolist() == [4, 7]
values[mask] = 0
assert values.tolist() == [1, 0, 0, 10]
```

Use `&`, `|`, and `~` with parenthesized comparisons for elementwise Boolean logic. Python's `and` and `or` expect single truth values. Use `mask.any()` or `mask.all()` when a single decision is needed.

`a[[0, 1], [1, 2]]` selects paired coordinates, not every combination. `a[np.ix_([0, 1], [1, 2])]` selects the cross-product submatrix.

## Shapes and broadcasting

```python
values = np.arange(6)
matrix = values.reshape(2, 3)
assert matrix.T.shape == (3, 2)
assert values[:, None].shape == (6, 1)
assert values[None, :].shape == (1, 6)
assert matrix.reshape(-1).shape == (6,)
```

`reshape` changes shape while preserving element count; one `-1` dimension can be inferred. It may return a view or require a copy. `ravel` returns a flattened view where possible; `flatten` returns a copy. A 1D array's transpose is still 1D: use `[:, None]` to create a column dimension.

Broadcasting compares dimensions from the right. Each pair must be equal or one must be `1`; missing leading axes act like size one. This rule avoids explicitly repeating operands, though the resulting output can still be large. See [broadcasting rules](https://numpy.org/doc/stable/user/basics.broadcasting.html).

```python
features = np.array([[1., 10., 100.], [2., 20., 200.]])
offsets = np.array([1., 2., 3.])
assert (features + offsets).shape == (2, 3)

rows = np.array([1, 2])[:, None]
columns = np.array([10, 20, 30])[None, :]
assert (rows + columns).tolist() == [[11, 21, 31], [12, 22, 32]]
```

Shapes `(n, 1)` and `(n,)` broadcast to `(n, n)`, which is a frequent accidental memory explosion. Check shapes before arithmetic, especially in ML loss functions.

## Ufuncs, reductions, and missing values

Ufuncs such as `np.add`, `np.sqrt`, `np.exp`, and `np.maximum` perform elementwise operations. Reductions combine values along selected axes.

```python
matrix = np.array([[1., 2., 3.], [4., 5., 6.]])
np.testing.assert_allclose(matrix.sum(axis=0), [5, 7, 9])
np.testing.assert_allclose(matrix.sum(axis=1), [6, 15])
assert matrix.mean(axis=0, keepdims=True).shape == (1, 3)
np.testing.assert_allclose(np.sqrt([1, 4, 9]), [1, 2, 3])
```

`axis=0` reduces over rows in a 2D table and produces one value per column. `axis=1` reduces over columns and produces one per row. `keepdims=True` retains reduced axes as size one for later broadcasting.

| Need | Tools |
|---|---|
| Totals and moments | `sum`, `mean`, `var`, `std` |
| Position/quantiles | `median`, `quantile`, `percentile` |
| Extremes | `min`, `max`, `argmin`, `argmax` |
| Running values | `cumsum`, `cumprod` |
| Bounds | `clip`, `minimum`, `maximum` |
| Conditions | `where`, `select`, `nonzero` |
| Missing/nonfinite | `isnan`, `isfinite`, `nanmean`, `nanmedian` |

NumPy's variance/standard deviation default uses `ddof=0`; sample formulas often use `ddof=1`. This differs from common pandas defaults. Quantile and percentile parameters use fractions versus percentages respectively.

```python
values = np.array([1., np.nan, 3.])
assert np.isnan(values).tolist() == [False, True, False]
assert np.nanmean(values) == 2

numerator = np.array([4., 8., 12.])
denominator = np.array([2., 0., 3.])
result = np.divide(
    numerator, denominator,
    out=np.full_like(numerator, np.nan), where=denominator != 0,
)
np.testing.assert_allclose(result, [2., np.nan, 4.], equal_nan=True)
```

`np.where(condition, a / b, 0)` still evaluates `a / b` before `where` selects values. Ufunc `where=` with initialized `out=` can avoid invalid arithmetic in masked positions. NaN is not equal to itself; test with `isnan`, not `== np.nan`.

## Sorting, combining, and searching

```python
values = np.array([4, 1, 4, 2])
assert np.sort(values).tolist() == [1, 2, 4, 4]
assert values[np.argsort(values)].tolist() == [1, 2, 4, 4]
unique, counts = np.unique(values, return_counts=True)
assert unique.tolist() == [1, 2, 4]
assert counts.tolist() == [1, 1, 2]
assert np.searchsorted([1, 3, 5], 4) == 2

left = np.ones((2, 3))
right = np.zeros((2, 3))
assert np.concatenate([left, right], axis=0).shape == (4, 3)
assert np.stack([left, right], axis=0).shape == (2, 2, 3)
```

`concatenate` joins on an existing axis; `stack` adds a new axis. `split` requires compatible split sizes; `array_split` can distribute uneven sizes. `searchsorted` assumes sorted input. `argpartition` finds partition positions without fully sorting; sort the selected portion if order matters.

## Linear algebra

```python
coefficients = np.array([[3., 1.], [1., 2.]])
target = np.array([9., 8.])
solution = np.linalg.solve(coefficients, target)
np.testing.assert_allclose(coefficients @ solution, target)
np.testing.assert_allclose(solution, [2., 3.])

u, singular_values, vt = np.linalg.svd(coefficients)
np.testing.assert_allclose(u @ np.diag(singular_values) @ vt, coefficients)
```

`*` is elementwise multiplication; `@` is matrix multiplication with appropriate shape rules. Use `solve(A, b)` instead of explicitly computing `inv(A) @ b`. Use `lstsq` for least-squares problems, `eigh` for symmetric/Hermitian eigenproblems, and `norm` for vector/matrix norms.

Check conditioning with `np.linalg.cond` when numerical sensitivity matters. A tiny residual alone does not guarantee an accurate solution to an ill-conditioned problem. For sparse or specialized solvers, see [SciPy](scipy.md).

## Random numbers and reproducibility

```python
rng = np.random.default_rng(42)
samples = rng.normal(loc=0, scale=1, size=(4, 2))
indices = rng.choice(10, size=3, replace=False)
assert samples.shape == (4, 2)
assert len(set(indices.tolist())) == 3
assert rng.integers(0, 10, size=5).shape == (5,)
```

Use an explicit generator and pass it to helpers instead of repeatedly resetting global random state. `integers` uses a half-open interval by default. `shuffle` mutates its input; `permutation` produces a permuted result.

Record seeds, library versions, algorithm settings, and data versions. A seed alone does not guarantee identical results across changed code, parallel scheduling, or all future generator implementations. The [Generator reference](https://numpy.org/doc/stable/reference/random/generator.html) documents the modern API.

## Files and interoperability

```python
from io import BytesIO

buffer = BytesIO()
np.save(buffer, np.array([1., 2., 3.]), allow_pickle=False)
buffer.seek(0)
restored = np.load(buffer, allow_pickle=False)
np.testing.assert_array_equal(restored, [1., 2., 3.])
```

`.npy` stores a single array with dtype/shape; `.npz` stores multiple arrays. `savez_compressed` compresses an archive. `loadtxt` and `genfromtxt` handle suitable text data, but table libraries offer richer CSV workflows.

Use `mmap_mode="r"` with suitable on-disk `.npy` files for memory-mapped access. Do not enable pickle for untrusted object arrays. Conversion to/from pandas, Arrow, or GPU libraries may copy, change dtypes, or move data between devices; verify the boundary behavior.

## Practical feature preprocessing

Fit preprocessing statistics on training data only, then reuse them on validation/test data:

```python
train = np.array([[1., 10.], [2., 20.], [3., 30.]])
test = np.array([[4., 40.]])
mean = train.mean(axis=0)
scale = train.std(axis=0)
scale = np.where(scale == 0, 1., scale)
train_scaled = (train - mean) / scale
test_scaled = (test - mean) / scale
np.testing.assert_allclose(train_scaled.mean(axis=0), [0, 0], atol=1e-12)
assert test_scaled.shape == (1, 2)
```

This assumes finite numeric values and uses population standard deviation. Decide how missing values and constant columns should be handled before fitting a model. Production ML pipelines help keep these transformations consistent across splits.

## Advanced topics, pitfalls, and practice

Learn strides and contiguous layout before optimizing memory access. `np.ascontiguousarray` can create a contiguous copy when required by another library. `einsum` expresses tensor contractions; validate shapes on small examples before optimizing. Broadcasting and vectorization can allocate huge intermediates; chunk when needed.

`np.vectorize` is primarily a convenience wrapper around Python calls, not a promise of compiled speed. Prefer built-in ufuncs, reductions, and suitable compiled algorithms. Avoid repeated `np.append` in a loop; collect then concatenate or preallocate.

Use `np.testing.assert_array_equal` for exact arrays and `assert_allclose` with justified tolerances for floating point. Specify absolute tolerance near zero; default relative tolerances are not appropriate for every scale.

Practice: implement row normalization with zero-row handling; compare view and copy mutation; compute pairwise distances and estimate intermediate memory; solve a least-squares regression; reproduce random sampling by passing a generator.

Reference hub: [NumPy user guide](https://numpy.org/doc/stable/user/index.html) and [API reference](https://numpy.org/doc/stable/reference/index.html).
