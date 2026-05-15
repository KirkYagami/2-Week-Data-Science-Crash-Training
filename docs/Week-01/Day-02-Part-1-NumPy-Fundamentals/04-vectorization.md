# Vectorization
## Writing Fast Code Without Writing Loops

The first time someone replaces a 50-line loop with a single NumPy expression and watches it run 200 times faster, something changes. Vectorization is not a trick — it is a different way of thinking about computation. Instead of writing "do this for each element," you write "apply this operation to the whole structure." The shift from loop-thinking to vector-thinking is one of the most valuable things you will develop as a data scientist.

## Learning Objectives

- Replace explicit Python for-loops with vectorized array operations
- Understand why vectorization is fast at the hardware level
- Use NumPy's universal functions (ufuncs) for element-wise math
- Aggregate arrays along specific axes and understand what that means geometrically
- Use `keepdims` to preserve dimensionality for broadcasting-compatible shapes
- Apply vectorization to realistic data science patterns: normalization, distance, returns

---

## Why Loops Are Slow

When Python executes a for-loop, each iteration goes through the Python interpreter. Each step involves:
- Looking up the variable name in a dictionary
- Checking the type of the object
- Dispatching to the appropriate C function
- Incrementing the reference count
- Returning a new Python object

For 1 million elements, that is 1 million round-trips through the interpreter overhead.

NumPy bypasses all of this. When you write `arr * 2`, NumPy calls a single C function that processes the entire array in one pass, using SIMD instructions to compute multiple values per CPU clock cycle.

```python
import numpy as np
import time

n = 2_000_000

data_list = list(range(n))
data_array = np.arange(n, dtype=np.float64)

# Python loop: square every element
start = time.perf_counter()
result = [x ** 2 for x in data_list]
loop_time = time.perf_counter() - start

# NumPy: square the whole array
start = time.perf_counter()
result = data_array ** 2
numpy_time = time.perf_counter() - start

print(f"Loop:   {loop_time * 1000:.1f} ms")
print(f"NumPy:  {numpy_time * 1000:.1f} ms")
print(f"Speedup: {loop_time / numpy_time:.0f}x")
# Output:
# Loop:   164.3 ms
# NumPy:    1.2 ms
# Speedup: 137x
```

> [!info] Where the Speed Actually Comes From
> Three mechanisms stack:
> 1. **No interpreter overhead** — computation happens in compiled C, not Python
> 2. **Cache locality** — contiguous memory means the CPU prefetcher loads the next values before they are needed
> 3. **SIMD (Single Instruction, Multiple Data)** — modern CPUs execute the same instruction on 4, 8, or 16 values simultaneously using AVX/SSE vector registers
>
> These three together explain the 100–300x speedup for embarrassingly parallel operations.

---

## Element-wise Arithmetic

Any arithmetic operator applied to a NumPy array operates on every element. No loop, no `map()`, no list comprehension needed.

```python
import numpy as np

arr = np.array([1.0, 2.0, 3.0, 4.0, 5.0])

# Scalar operations — scalar is applied to every element
print(arr + 10)    # Output: [11. 12. 13. 14. 15.]
print(arr * 3)     # Output: [ 3.  6.  9. 12. 15.]
print(arr ** 2)    # Output: [ 1.  4.  9. 16. 25.]
print(arr / 2)     # Output: [0.5 1.  1.5 2.  2.5]
print(arr % 3)     # Output: [1. 2. 0. 1. 2.]
print(1 / arr)     # Output: [1.    0.5   0.333 0.25  0.2  ]
```

When two arrays have the same shape, operations run element-pair by element-pair:

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([10, 20, 30, 40, 50])

print(a + b)   # Output: [11 22 33 44 55]
print(a * b)   # Output: [10 40 90 160 250]
print(b / a)   # Output: [10. 10. 10. 10. 10.]
print(a ** b)  # Output: [1, 1048576, ...]  ← a[i] raised to b[i]
```

### 2-D Operations

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])

B = np.array([[10, 20, 30],
              [40, 50, 60]])

print(A + B)
# Output:
# [[11 22 33]
#  [44 55 66]]

print(A * B)   # element-wise, NOT matrix multiplication
# Output:
# [[ 10  40  90]
#  [160 250 360]]
```

> [!warning] `*` is Element-wise, `@` is Matrix Multiply
> ```python
> A = np.array([[1, 2], [3, 4]])
> B = np.array([[5, 6], [7, 8]])
>
> print(A * B)   # element-wise: [[5, 12], [21, 32]]
> print(A @ B)   # matrix multiply: [[19, 22], [43, 50]]
>
> # These are completely different operations.
> # When working with weights and activations in neural nets, you always want @.
> # When scaling features element-by-element, you want *.
> ```

---

## Universal Functions (ufuncs)

Universal functions are NumPy functions that operate element-wise on arrays. They are implemented in C and are the backbone of NumPy's vectorization.

```python
import numpy as np

arr = np.array([0.0, 1.0, 2.0, 3.0, 4.0])

# Trigonometry
print(np.sin(arr))     # Output: [0.    0.841 0.909 0.141 -0.757]
print(np.cos(arr))     # Output: [1.    0.540 -0.416 -0.990 -0.654]

# Working with degrees
degrees = np.linspace(0, 360, 5)
print(np.sin(np.radians(degrees)).round(3))
# Output: [ 0.  1.  0. -1.  0.]

# Exponential and logarithm
print(np.exp(arr))         # Output: [ 1.     2.718  7.389 20.086 54.598]  ← e^x
print(np.log(arr + 0.01))  # natural log — avoid log(0) with offset
print(np.log2(arr + 1))    # log base 2
print(np.log10(arr + 1))   # log base 10

# Roots and powers
print(np.sqrt(arr))           # Output: [0.    1.    1.414 1.732 2.   ]
print(np.power(arr, 3))       # Output: [ 0.  1.  8. 27. 64.]
print(np.abs(arr - 2.5))      # Output: [2.5  1.5  0.5  0.5  1.5]

# Rounding
vals = np.array([1.2, 1.5, 1.8, 2.5, -1.5])
print(np.round(vals))   # Output: [ 1.  2.  2.  2. -2.]  ← banker's rounding
print(np.floor(vals))   # Output: [ 1.  1.  1.  2. -2.]  ← always down
print(np.ceil(vals))    # Output: [ 2.  2.  2.  3. -1.]  ← always up
print(np.trunc(vals))   # Output: [ 1.  1.  1.  2. -1.]  ← toward zero
```

> [!info] Banker's Rounding
> `np.round(2.5)` gives `2.0`, not `3.0`. NumPy uses banker's rounding (round half to even), which reduces cumulative rounding error in statistical computations. If you need standard rounding (half up), use `np.floor(x + 0.5)`.

### Min/Max Element-wise Functions

```python
import numpy as np

a = np.array([3, 7, 2, 9, 1])
b = np.array([5, 4, 8, 1, 6])

# Element-wise max/min between two arrays
print(np.maximum(a, b))  # Output: [5 7 8 9 6]  ← max of each pair
print(np.minimum(a, b))  # Output: [3 4 2 1 1]  ← min of each pair

# Clip: enforce a value range
sensor_readings = np.array([-10, 5, 120, 85, -3, 100, 200])
valid = np.clip(sensor_readings, 0, 100)
print(valid)  # Output: [  0   5 100  85   0 100 100]
```

---

## Aggregation Functions

Aggregation reduces an array (or a dimension of it) to a summary value. These are the NumPy equivalents of SQL's `SUM`, `AVG`, `MAX`, `MIN`.

```python
import numpy as np

data = np.array([4, 7, 2, 9, 1, 5, 8, 3, 6, 10])

print(np.sum(data))      # Output: 55
print(np.mean(data))     # Output: 5.5
print(np.median(data))   # Output: 5.5
print(np.std(data))      # Output: 2.872...  ← population std
print(np.var(data))      # Output: 8.25      ← population variance
print(np.min(data))      # Output: 1
print(np.max(data))      # Output: 10
print(np.argmin(data))   # Output: 4         ← index of minimum value
print(np.argmax(data))   # Output: 9         ← index of maximum value

# Cumulative operations
print(np.cumsum(np.array([1, 2, 3, 4])))    # Output: [ 1  3  6 10]
print(np.cumprod(np.array([1, 2, 3, 4])))   # Output: [  1   2   6  24]

# Percentiles
print(np.percentile(data, 25))   # Output: 3.25   ← Q1
print(np.percentile(data, 75))   # Output: 7.75   ← Q3

# Sorting
print(np.sort(data))     # Output: [ 1  2  3  4  5  6  7  8  9 10]
print(np.argsort(data))  # Output: indices that would sort the array
```

---

## Axis-wise Operations

For multi-dimensional arrays, you choose which dimension to aggregate **along**. The key mental model: the axis you specify is the one that **gets collapsed**.

```python
import numpy as np

# Sales data: 3 products × 4 months
sales = np.array([[120, 140, 90, 110],   # product 0
                  [200, 180, 220, 195],   # product 1
                  [80,  95,  70,  85]])   # product 2

print(sales.shape)   # Output: (3, 4)

# Total without axis: sum everything
print(np.sum(sales))            # Output: 1585  ← grand total

# axis=0: collapse rows → one result per column (monthly totals)
print(np.sum(sales, axis=0))    # Output: [400 415 380 390]
print(np.mean(sales, axis=0))   # Output: [133.3 138.3 126.7 130. ]

# axis=1: collapse columns → one result per row (product totals)
print(np.sum(sales, axis=1))    # Output: [460 795 330]
print(np.mean(sales, axis=1))   # Output: [115.  198.75  82.5]
```

**Visual:**

```
sales = [[120, 140,  90, 110],   axis=0: "downward" (collapse rows)
         [200, 180, 220, 195],      ↓     ↓     ↓     ↓
         [ 80,  95,  70,  85]]   → [400, 415, 380, 390]

axis=1: "rightward" (collapse columns)
  [120, 140,  90, 110] →  460
  [200, 180, 220, 195] →  795
  [ 80,  95,  70,  85] →  330
```

> [!tip] Remembering Axis Direction
> The axis you name is the one that **disappears**. `axis=0` means "go down the rows" — rows disappear, you get one value per column. `axis=1` means "go across the columns" — columns disappear, you get one value per row.

### `keepdims` — Preserve the Dimension

When you aggregate with `axis`, the result loses a dimension. `keepdims=True` keeps it as a size-1 dimension, which is essential for broadcasting.

```python
import numpy as np

data = np.array([[10, 20, 30],
                 [40, 50, 60]])

# Without keepdims: shape goes from (2, 3) to (3,)
col_means = data.mean(axis=0)
print(col_means.shape)   # Output: (3,)

# With keepdims: shape goes from (2, 3) to (1, 3)
col_means_k = data.mean(axis=0, keepdims=True)
print(col_means_k.shape)  # Output: (1, 3)

# Now you can subtract the column mean from every row (broadcasting works)
centered = data - col_means_k
print(centered)
# Output:
# [[-15. -15. -15.]
#  [ 15.  15.  15.]]
```

---

## Statistical Operations

```python
import numpy as np

data = np.array([2.1, 3.5, 1.8, 4.2, 3.0, 5.1, 2.7, 4.8, 3.3, 2.9])

# Descriptive statistics
print(f"Mean:   {data.mean():.3f}")         # Output: 3.340
print(f"Median: {np.median(data):.3f}")     # Output: 3.150
print(f"Std:    {data.std():.3f}")          # Output: 0.963
print(f"Range:  {data.max() - data.min():.3f}")  # Output: 3.300

# Z-score standardization: (value - mean) / std
z = (data - data.mean()) / data.std()
print(z.round(2))
# Output: [-1.28  0.16 -1.59  0.89 -0.35  1.83 -0.66  1.51 -0.04 -0.46]
print(f"Z-score mean: {z.mean():.6f}")  # Output: ~0.0
print(f"Z-score std:  {z.std():.6f}")   # Output: ~1.0
```

---

## Linear Algebra

```python
import numpy as np

A = np.array([[1, 2],
              [3, 4]])
B = np.array([[5, 6],
              [7, 8]])

# Matrix multiplication
print(A @ B)
# Output:
# [[19 22]
#  [43 50]]

# Dot product of vectors
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])
print(np.dot(u, v))   # Output: 32  ← 1*4 + 2*5 + 3*6

# Transpose
print(A.T)
# Output:
# [[1 3]
#  [2 4]]

# Inverse and determinant
print(np.linalg.det(A))     # Output: -2.0
print(np.linalg.inv(A))
# Output:
# [[-2.   1. ]
#  [ 1.5 -0.5]]

# Solve system Ax = b
b = np.array([5, 11])
x = np.linalg.solve(A, b)
print(x)  # Output: [1. 2.]  ← A @ x == b
```

---

## Real-World Vectorization Patterns

These patterns appear constantly in data science. Knowing the vectorized form saves you both time and memory.

### Min-Max Normalization

```python
import numpy as np

data = np.array([20.0, 35.0, 10.0, 50.0, 45.0, 15.0, 30.0])

# Scale all values to the range [0, 1]
scaled = (data - data.min()) / (data.max() - data.min())
print(scaled.round(3))
# Output: [0.25  0.625 0.    1.    0.875 0.125 0.5  ]
```

### Euclidean Distance Between Two Points

```python
import numpy as np

point_a = np.array([1.0, 2.0, 3.0])
point_b = np.array([4.0, 6.0, 3.0])

# Manual vectorized version
distance = np.sqrt(np.sum((point_a - point_b) ** 2))
print(distance)  # Output: 5.0

# NumPy built-in
distance = np.linalg.norm(point_a - point_b)
print(distance)  # Output: 5.0
```

### Daily Returns for a Time Series

```python
import numpy as np

prices = np.array([100.0, 103.5, 101.2, 107.8, 105.3, 110.0])

# Return[i] = (price[i] - price[i-1]) / price[i-1]
# np.diff computes consecutive differences
returns = np.diff(prices) / prices[:-1] * 100
print(returns.round(2))
# Output: [ 3.5  -2.22  6.52 -2.32  4.46]
```

### Batch Matrix Computation

```python
import numpy as np

# 100 data points with 5 features each
rng = np.random.default_rng(42)
X = rng.standard_normal((100, 5))

# Compute per-feature mean and std in one pass
means = X.mean(axis=0)        # shape (5,)
stds  = X.std(axis=0)         # shape (5,)

# Standardize all features simultaneously (broadcasting)
X_standardized = (X - means) / stds   # (100, 5) - (5,) → (100, 5)
print(X_standardized.mean(axis=0).round(6))  # Output: [~0. ~0. ~0. ~0. ~0.]
print(X_standardized.std(axis=0).round(6))   # Output: [~1. ~1. ~1. ~1. ~1.]
```

---

## Comparison and Logic on Arrays

```python
import numpy as np

a = np.array([1, 2, 3, 4, 5])
b = np.array([3, 2, 5, 1, 5])

# Element-wise comparisons → boolean arrays
print(a == b)   # Output: [False  True False False  True]
print(a < b)    # Output: [ True False  True False False]
print(a >= b)   # Output: [False  True False  True  True]

# Array-level logic
arr = np.array([True, True, False])
print(np.all(arr))   # Output: False  ← are ALL True?
print(np.any(arr))   # Output: True   ← is ANY True?

# Practical use
scores = np.array([85, 72, 91, 63, 88])
print(np.all(scores >= 60))   # Output: True   ← everyone passed
print(np.any(scores < 70))    # Output: True   ← someone is struggling
print(np.sum(scores >= 80))   # Output: 3      ← count of A students
print(np.mean(scores >= 80))  # Output: 0.6    ← fraction of A students
```

> [!tip] Boolean Arrays as Integers
> NumPy treats `True` as 1 and `False` as 0 in arithmetic. So `np.sum(arr > threshold)` counts the elements that meet the condition, and `np.mean(arr > threshold)` gives the fraction. This is a very common pattern.

---

> [!success] Key Takeaways
> - Vectorization replaces for-loops with array operations that run in compiled C at SIMD speed
> - Every arithmetic operator (+, -, *, /, **, %) is element-wise on arrays
> - `*` is element-wise multiply; `@` is matrix multiply — these are different things
> - ufuncs (`np.sin`, `np.exp`, `np.sqrt`, etc.) apply element-wise at C speed
> - `axis=0` collapses rows (result per column); `axis=1` collapses columns (result per row)
> - `keepdims=True` preserves the dimension as size-1 — essential for broadcasting later
> - `np.sum(condition)` counts; `np.mean(condition)` gives the fraction

---

[[03-indexing-and-slicing]] | [[05-broadcasting]]
