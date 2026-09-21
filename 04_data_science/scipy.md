# SciPy: Scientific Algorithms

> **Purpose:** Choose and use numerical algorithms for optimization, statistics, integration, interpolation, sparse systems, signals, and spatial data. Examples run sequentially and use small synthetic inputs.

## Contents

- [Setup and module map](#setup-and-module-map)
- [Optimization and roots](#optimization-and-roots)
- [Curve fitting](#curve-fitting)
- [Integration and differential equations](#integration-and-differential-equations)
- [Interpolation](#interpolation)
- [Statistics and distributions](#statistics-and-distributions)
- [Linear algebra and sparse arrays](#linear-algebra-and-sparse-arrays)
- [Signal processing and Fourier transforms](#signal-processing-and-fourier-transforms)
- [Spatial algorithms and images](#spatial-algorithms-and-images)
- [Numerical reliability and practice](#numerical-reliability-and-practice)

## Setup and module map

```bash
python -m pip install scipy
```

```python
import numpy as np
from scipy import optimize, integrate, interpolate, stats, linalg, sparse, signal, fft, spatial, ndimage, special
```

NumPy supplies arrays and core numerical operations; SciPy supplies a broader collection of scientific algorithms built around arrays. It is not a general replacement for an ML framework.

| Module | Common applications |
|---|---|
| `optimize` | Minimize objectives, solve roots, fit parameters |
| `integrate` | Quadrature and initial-value differential equations |
| `interpolate` | Estimate values between observations |
| `stats` | Distributions, tests, descriptive statistics, resampling |
| `linalg` | Dense matrix factorizations and solvers |
| `sparse` / `sparse.linalg` | Sparse arrays and sparse solvers |
| `signal` | Filtering, peaks, convolution, spectral analysis |
| `fft` | Fourier transforms and frequencies |
| `spatial` | Distances, nearest-neighbor trees, geometric algorithms |
| `ndimage` | N-dimensional image operations |
| `special` | Numerically specialized mathematical functions |
| `cluster` | Clustering building blocks |
| `io` | Scientific file formats |
| `constants` | Physical/mathematical constants |

The [SciPy user guide](https://docs.scipy.org/doc/scipy/tutorial/index.html) links specialized tutorials. Import the relevant submodule explicitly instead of assuming every operation lives at the top level.

## Optimization and roots

Optimization finds inputs minimizing an objective. Root finding finds inputs where a function is zero; these are related but distinct tasks.

```python
def objective(point):
    return (point[0] - 3) ** 2 + 2 * (point[1] + 1) ** 2


def gradient(point):
    return np.array([2 * (point[0] - 3), 4 * (point[1] + 1)])


result = optimize.minimize(objective, x0=[0., 0.], jac=gradient, method="BFGS")
assert result.success, result.message
np.testing.assert_allclose(result.x, [3, -1], atol=1e-5)

root = optimize.root_scalar(lambda x: x * x - 2, bracket=[0, 2], method="brentq")
assert root.converged
np.testing.assert_allclose(root.root, np.sqrt(2))
```

Inspect convergence status, message, objective value, constraints, and residuals. Returning a result object does not mean the algorithm succeeded. A local optimizer does not guarantee a global minimum.

| Situation | Starting point |
|---|---|
| Smooth unconstrained objective | BFGS; supply derivatives when available |
| Bounds on variables | L-BFGS-B or a suitable bounded method |
| General constraints | SLSQP or trust-constr, depending on the problem |
| Scalar root with a sign-changing bracket | `root_scalar(..., method="brentq")` |
| Vector of residuals | `least_squares` |
| Linear programming | `linprog` |
| Global search | `differential_evolution` or another suitable strategy |

Scale variables and objectives sensibly. Poor scaling, noisy objectives, incorrect gradients, and weak initial guesses can break convergence. Verify a provided derivative against finite differences on small inputs. Method-specific options are documented in [minimize](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html).

## Curve fitting

```python
def line(x, slope, intercept):
    return slope * x + intercept


x = np.linspace(0, 4, 9)
y = line(x, 2., 1.)
parameters, covariance = optimize.curve_fit(line, x, y)
np.testing.assert_allclose(parameters, [2, 1], atol=1e-8)
```

`curve_fit` estimates model parameters. Inspect residuals, parameter identifiability, units, bounds, and the effect of initial guesses. Its covariance estimate depends on assumptions and scaling; it is not automatically a trustworthy uncertainty statement for every nonlinear or misspecified model.

Use `least_squares` for more direct control over residuals, robust losses, and bounds. Do not judge a fit only by its training residual; validate the model on meaningful held-out data when prediction is the goal.

## Integration and differential equations

```python
area, estimated_error = integrate.quad(lambda x: x * x, 0, 1)
np.testing.assert_allclose(area, 1 / 3)
assert estimated_error >= 0

solution = integrate.solve_ivp(
    fun=lambda time, state: -0.5 * state,
    t_span=(0, 4), y0=[2.], t_eval=np.linspace(0, 4, 9),
    rtol=1e-8, atol=1e-10,
)
assert solution.success, solution.message
np.testing.assert_allclose(solution.y[0], 2 * np.exp(-0.5 * solution.t), rtol=1e-7)
```

`quad` integrates a callable adaptively and returns an estimated error, not a universal rigorous bound. `trapezoid` and `simpson` integrate sampled data. Sampling density, discontinuities, and singularities affect reliability.

`solve_ivp` solves an initial-value ODE. `t_eval` chooses output points; the solver can take different internal steps. Choose a method suitable for stiffness, set tolerances in relation to variable scales, and inspect solver messages. Events can locate threshold crossings; conservation laws can provide additional checks.

## Interpolation

```python
x = np.array([0., 1., 2., 3.])
y = np.array([0., 1., 1.5, 2.])
curve = interpolate.PchipInterpolator(x, y, extrapolate=False)
np.testing.assert_allclose(curve(x), y)
assert np.isnan(curve(-1))
```

Interpolation estimates values within the observed range; extrapolation goes beyond it and can be unreliable. Cubic splines can be smooth but overshoot; PCHIP preserves monotonicity for monotone data. `RegularGridInterpolator` works on regular multidimensional grids; scattered data needs different tools.

Check sorted coordinates, duplicates, gaps, units, and boundary behavior. Interpolation is not denoising, causal forecasting, or evidence that the interpolated curve is physically correct.

## Statistics and distributions

### Distribution operations

```python
normal = stats.norm(loc=10, scale=2)
np.testing.assert_allclose(normal.cdf(10), 0.5)
np.testing.assert_allclose(normal.ppf(0.5), 10)
assert normal.sf(14) > 0
rng = np.random.default_rng(42)
samples = normal.rvs(size=20, random_state=rng)
assert samples.shape == (20,)
```

| Method | Meaning |
|---|---|
| `pdf` / `pmf` | Density / discrete probability mass |
| `cdf` | Cumulative probability |
| `sf` | Upper-tail probability; often more accurate than `1 - cdf` in tails |
| `ppf` | Quantile function |
| `rvs` | Random draws |
| `logpdf`, `logpmf` | Log probabilities for numerical stability |

A continuous density at one point is not the probability of exactly that point. Confirm parameterization: for a normal distribution, `scale` is standard deviation, not variance.

### Comparing independent means

```python
group_a = np.array([10., 12., 11., 13., 9., 10.])
group_b = np.array([14., 13., 15., 12., 16., 14.])
test = stats.ttest_ind(group_a, group_b, equal_var=False, nan_policy="raise")
assert 0 <= test.pvalue <= 1
assert test.statistic < 0
difference = group_a.mean() - group_b.mean()
assert difference < 0
```

`equal_var=False` selects Welch's independent-sample t-test. Use a paired analysis for paired observations. Independence, sampling design, distributional suitability, and outliers still matter; a function call cannot verify those assumptions. See [ttest_ind](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_ind.html).

A p-value is not the probability that the null hypothesis is true, and statistical significance is not effect size or practical importance. Report the estimated effect, uncertainty, sample sizes, and study assumptions. Correct for multiple testing when the analysis calls for it.

### Other statistical tools

| Goal | Candidate tools | Check |
|---|---|---|
| Paired mean comparison | `ttest_rel` | Correct pairing and difference assumptions |
| Rank-based two-group analysis | `mannwhitneyu`, `wilcoxon` | Independent versus paired design; hypotheses differ from a generic median test |
| Categorical association | `chi2_contingency`, `fisher_exact` | Expected counts and sampling design |
| Correlation | `pearsonr`, `spearmanr` | Linear versus monotonic association; correlation is not causation |
| Resampling intervals | `bootstrap` | Resampling unit and dependence structure |
| Randomization inference | `permutation_test` | Exchangeability under the null |

Do not use a normality test as an automatic switch that chooses every downstream method. Understand the scientific question and the estimator first.

## Linear algebra and sparse arrays

```python
matrix = np.array([[4., 1.], [1., 3.]])
rhs = np.array([1., 2.])
answer = linalg.solve(matrix, rhs, assume_a="pos")
np.testing.assert_allclose(matrix @ answer, rhs)

from scipy.sparse.linalg import spsolve

compressed = sparse.csr_array(matrix)
sparse_answer = spsolve(compressed, rhs)
np.testing.assert_allclose(sparse_answer, answer)
assert compressed.nnz == 4
```

`assume_a="pos"` asserts positive definiteness; do not set it unless the matrix has that property. Sparse representations save space only when sufficiently few entries are stored.

| Format | Typical strength |
|---|---|
| COO | Build from coordinate triplets |
| CSR | Row-oriented operations and matrix-vector multiplication |
| CSC | Column-oriented operations and many factorization workflows |
| LIL / DOK | Incremental construction |

For modern sparse arrays, use `@` for matrix multiplication and `*` for elementwise multiplication. Legacy sparse matrix classes have different historical operator behavior. Avoid accidental `.toarray()` on a huge sparse object. Factorization can create fill-in, consuming more memory than the input sparsity suggests.

## Signal processing and Fourier transforms

```python
sample_rate = 100.
time = np.arange(100) / sample_rate
wave = np.sin(2 * np.pi * 5 * time)
spectrum = np.abs(fft.rfft(wave))
frequencies = fft.rfftfreq(len(wave), d=1 / sample_rate)
assert frequencies[np.argmax(spectrum)] == 5

peaks, properties = signal.find_peaks([0, 1, 0, 2, 0], height=0.5)
assert peaks.tolist() == [1, 3]
```

The sampling rate sets the frequency scale. Aliasing, windowing, leakage, normalization, and finite sample length affect interpretation. A Fourier magnitude is not automatically a calibrated power spectral density.

Use `signal.welch` for suitable PSD estimation, `convolve` for convolution, and `butter(..., output="sos")` with SOS filtering for many stable digital-filter workflows. `sosfiltfilt` uses future and past samples, so it is not an online causal filter. Edge behavior and short signals need attention.

## Spatial algorithms and images

```python
points = np.array([[0., 0.], [2., 0.], [0., 3.]])
tree = spatial.cKDTree(points)
distance, index = tree.query([1.8, 0.1])
assert index == 1
assert distance < 0.3

image = np.zeros((7, 7))
image[3, 3] = 1
blurred = ndimage.gaussian_filter(image, sigma=1)
assert blurred.shape == image.shape
assert 0 < blurred[3, 3] < 1
```

Distance metrics must match feature scales and geometry. Euclidean distances on raw latitude/longitude are not generally meaningful geographic distances. Tree methods can degrade in high dimensions.

`ndimage` includes filtering, morphology, connected-component labeling, and geometric transformations. Boundary modes and interpolation order affect results. Keep channel axes distinct from spatial axes when filtering color images.

`special.expit` computes a stable logistic sigmoid; `special.logsumexp` helps avoid overflow/underflow when combining log probabilities.

```python
assert special.expit(0) == 0.5
assert np.isfinite(special.logsumexp([1000., 1001.]))
```

## Numerical reliability and practice

Check units, shapes, dtypes, finite values, algorithm assumptions, convergence, conditioning, and tolerances. Validate on a case with a known answer before applying an algorithm to unknown data. A plausible-looking plot is not sufficient numerical verification.

Practice: fit a noisy exponential curve and inspect residuals; compare interpolation outside observed bounds; solve a sparse linear system without densifying; identify frequencies after adding noise; simulate the sampling distribution of a mean; compare a paired and unpaired test on the same measurements and explain the design difference.

Reference: [SciPy API](https://docs.scipy.org/doc/scipy/reference/index.html). Related: [NumPy](numpy.md), [Matplotlib](matplotlib.md).
