# 04 — Vectorization
## Writing Fast Code Without Writing Loops

> [!quote] "The first rule of NumPy: if you're writing a for-loop over array elements, you're probably doing it wrong."

---

## 🧭 Table of Contents

- [[#What is Vectorization?]]
- [[#Element-wise Arithmetic]]
- [[#Universal Functions (ufuncs)]]
- [[#Aggregation Functions]]
- [[#Axis-wise Operations]]
- [[#Mathematical Functions]]
- [[#Comparison and Logic]]
- [[#String Vectorization]]
- [[#Performance Benchmarks]]
- [[#Summary]]

---

## What is Vectorization?

**Vectorization** is the process of replacing explicit loops with array operations that apply to **all elements at once**.

### The Loop vs Vector Mindset

```python
import numpy as np

data = np.array([1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0])

# ❌ Thinking in loops (slow, verbose)
result = np.empty(len(data))
for i in range(len(data)):
    result[i] = data[i] ** 2 + 2 * data[i] + 1

# ✅ Thinking in vectors (fast, clean)
result = data**2 + 2*data + 1

print(result)  # [  4.   9.  16.  25.  36.  49.  64.  81. 100. 121.]
```

### Why Vectorization is Faster

```
Loop approach (Python):
  Iteration 1: Python overhead → get data[0] → compute → store result[0]
  Iteration 2: Python overhead → get data[1] → compute → store result[1]
  ...                         (10 rounds of overhead for 10 elements)

Vectorized approach (NumPy):
  One C function call → process ALL elements simultaneously in compiled C code
  Zero Python loop overhead
```

> [!info] How Fast Is It?
> For an array of 1 million elements, a vectorized NumPy operation is typically **100-1000x faster** than an equivalent Python loop.

---

## Element-wise Arithmetic

When you apply an arithmetic operator between a NumPy array and a scalar (or two arrays of the same shape), the operation is applied to **every element**:

### Array + Scalar

```python
arr = np.array([1, 2, 3, 4, 5])

# Every operation broadcasts the scalar to all elements
print(arr + 10)   # [11 12 13 14 15]  ← add 10 to each
print(arr - 3)    # [-2 -1  0  1  2]  ← subtract 3 from each
print(arr * 4)    # [ 4  8 12 16 20]  ← multiply each by 4
print(arr / 2)    # [0.5 1.  1.5 2.  2.5]  ← divide each by 2
print(arr // 2)   # [0 1 1 2 2]       ← floor division
print(arr % 3)    # [1 2 0 1 2]       ← modulo (remainder)
print(arr ** 2)   # [ 1  4  9 16 25]  ← each element squared
print(-arr)       # [-1 -2 -3 -4 -5]  ← negate each element
```

### Array + Array (Element-wise)

When two arrays have the **same shape**, operations are element-wise (index 0 with 0, index 1 with 1, etc.):

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([10, 20, 30, 40, 50])

print(a + b)   # [11 22 33 44 55]
print(a - b)   # [ -9 -18 -27 -36 -45]
print(a * b)   # [ 10  40  90 160 250]
print(a / b)   # [0.1 0.1 0.1 0.1 0.1]
print(a ** b)  # [1, 1048576, 2.05891...e+14, ...] ← a[i] ^ b[i]
```

### 2D Element-wise Operations

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])

B = np.array([[10, 20, 30],
              [40, 50, 60]])

print(A + B)
# [[11 22 33]
#  [44 55 66]]

print(A * B)   # Element-wise multiply (NOT matrix multiply!)
# [[ 10  40  90]
#  [160 250 360]]

# Matrix multiplication uses @ operator or np.dot()
print(A @ A.T)     # Matrix multiply A by its transpose
print(np.dot(A, B.T))  # Same thing with np.dot
```

> [!warning] `*` is NOT Matrix Multiplication!
> `A * B` = element-wise multiplication (each element with its pair)
> `A @ B` = matrix multiplication (dot product of rows and columns)
> These are **completely different** operations!

---

## Universal Functions (ufuncs)

**Universal Functions (ufuncs)** are functions that operate element-wise on arrays at C speed. They are the heart of NumPy's vectorization.

### Math ufuncs

```python
arr = np.array([0.0, 1.0, 2.0, 3.0, 4.0])

# Trigonometry
print(np.sin(arr))    # [0.    0.841 0.909 0.141 -0.757]
print(np.cos(arr))    # [1.    0.540 -0.416 -0.990 -0.654]
print(np.tan(arr))

# Trig with degrees
degrees = np.array([0, 30, 45, 60, 90])
radians = np.radians(degrees)
print(np.sin(radians))  # [0.  0.5  0.707  0.866  1.0]

# Exponential and logarithm
print(np.exp(arr))        # [1.  2.718 7.389 20.09 54.6]  ← e^x
print(np.exp2(arr))       # [1.  2.   4.    8.   16.]     ← 2^x
print(np.log(arr + 0.1))  # Natural log (log base e)
print(np.log2(arr + 1))   # Log base 2
print(np.log10(arr + 1))  # Log base 10

# Power and roots
print(np.sqrt(arr))    # [0.   1.   1.414 1.732 2.   ]
print(np.cbrt(arr))    # Cube root
print(np.power(arr, 3))  # [0.  1.  8.  27.  64.]

# Absolute value
print(np.abs(np.array([-3, -1, 0, 2, 4])))  # [3 1 0 2 4]
```

### Rounding ufuncs

```python
arr = np.array([1.1, 1.5, 1.9, 2.5, -1.5, -1.9])

print(np.round(arr))    # [ 1.  2.  2.  2. -2. -2.]  ← nearest even (banker's rounding)
print(np.floor(arr))    # [ 1.  1.  1.  2. -2. -2.]  ← round down
print(np.ceil(arr))     # [ 2.  2.  2.  3. -1. -1.]  ← round up
print(np.trunc(arr))    # [ 1.  1.  1.  2. -1. -1.]  ← truncate toward zero
```

### Min/Max ufuncs

```python
a = np.array([1, 5, 3, 8, 2])
b = np.array([4, 2, 7, 1, 6])

# Element-wise max/min of two arrays
print(np.maximum(a, b))  # [4 5 7 8 6]  ← max of each pair
print(np.minimum(a, b))  # [1 2 3 1 2]  ← min of each pair

# Clip (limit values to a range)
arr = np.array([-5, -1, 0, 3, 7, 15])
print(np.clip(arr, 0, 10))  # [0 0 0 3 7 10]  ← values clamped to [0, 10]
```

---

## Aggregation Functions

Aggregation functions **reduce** an array to a single value (or smaller array):

```python
arr = np.array([4, 7, 2, 9, 1, 5, 8, 3, 6, 10])

# Basic aggregations
print(np.sum(arr))      # 55
print(np.min(arr))      # 1
print(np.max(arr))      # 10
print(np.mean(arr))     # 5.5
print(np.median(arr))   # 5.5
print(np.std(arr))      # Standard deviation: ~2.87
print(np.var(arr))      # Variance: ~8.25
print(np.prod(arr))     # Product of all elements

# Index of min/max
print(np.argmin(arr))   # 4  ← index of the minimum value
print(np.argmax(arr))   # 9  ← index of the maximum value

# Sorting
print(np.sort(arr))     # [ 1  2  3  4  5  6  7  8  9 10]
print(np.argsort(arr))  # [4 2 7 0 5 8 1 6 3 9]  ← indices that would sort

# Cumulative operations
print(np.cumsum(arr))   # [ 4 11 13 22 23 28 36 39 45 55]
print(np.cumprod(np.array([1,2,3,4,5])))  # [1 2 6 24 120]

# Percentiles
print(np.percentile(arr, 25))  # 3.25   ← 25th percentile (Q1)
print(np.percentile(arr, 50))  # 5.5    ← 50th percentile (Q2/median)
print(np.percentile(arr, 75))  # 7.75   ← 75th percentile (Q3)
```

---

## Axis-wise Operations

For multi-dimensional arrays, you can specify **which axis** to aggregate along:

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
#                 col0 col1 col2

# No axis = aggregate everything (returns scalar)
print(np.sum(matrix))       # 45  ← sum of all 9 elements

# axis=0 = collapse rows → result has shape (cols,)
print(np.sum(matrix, axis=0))   # [12 15 18]  ← sum of each column
print(np.mean(matrix, axis=0))  # [4.  5.  6.]  ← mean of each column
print(np.max(matrix, axis=0))   # [7 8 9]  ← max of each column

# axis=1 = collapse columns → result has shape (rows,)
print(np.sum(matrix, axis=1))   # [ 6 15 24]  ← sum of each row
print(np.mean(matrix, axis=1))  # [2.  5.  8.]  ← mean of each row
print(np.max(matrix, axis=1))   # [3 6 9]  ← max of each row
```

### 🧠 Axis Direction Visual

```
matrix = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]

axis=0 (operates "downward", across rows):
   ↓  ↓  ↓
  [1, 2, 3]
  [4, 5, 6]
  [7, 8, 9]
   ↓  ↓  ↓
Result: [12, 15, 18]  (shape = (3,))

axis=1 (operates "rightward", across columns):
  [1, 2, 3] →  6
  [4, 5, 6] → 15
  [7, 8, 9] → 24
Result: [6, 15, 24]  (shape = (3,))
```

> [!tip] Memory Trick for Axis
> - `axis=0` = **"along rows"** (result has one value per **column**)
> - `axis=1` = **"along columns"** (result has one value per **row**)
> 
> Or just remember: the axis you specify is the one that **gets eliminated** in the output.

### keepdims Parameter

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

# Without keepdims — loses dimensionality
col_means = np.mean(matrix, axis=0)
print(col_means.shape)  # (3,)  ← 1D

# With keepdims — preserves shape for broadcasting
col_means = np.mean(matrix, axis=0, keepdims=True)
print(col_means.shape)  # (1, 3)  ← still 2D!
print(col_means)        # [[2.5 3.5 4.5]]

# This allows you to then normalize:
normalized = matrix / col_means  # Broadcasting works because shapes align
print(normalized)
```

---

## Mathematical Functions

### Statistical Functions

```python
data = np.array([2.1, 3.5, 1.8, 4.2, 3.0, 5.1, 2.7, 4.8, 3.3, 2.9])

# Descriptive statistics
print(f"Mean:   {np.mean(data):.3f}")     # 3.340
print(f"Median: {np.median(data):.3f}")   # 3.150
print(f"Std:    {np.std(data):.3f}")      # 0.986
print(f"Var:    {np.var(data):.3f}")      # 0.972
print(f"Range:  {np.ptp(data):.3f}")      # peak-to-peak = max - min = 3.300

# Z-score normalization (manual, using vectorization)
z_scores = (data - np.mean(data)) / np.std(data)
print(z_scores.round(2))
# [-1.26  0.16 -1.56  0.87 -0.34  1.78 -0.65  1.48 -0.04 -0.44]
```

### Linear Algebra

```python
A = np.array([[1, 2],
              [3, 4]])

B = np.array([[5, 6],
              [7, 8]])

# Matrix multiplication
print(A @ B)          # or np.matmul(A, B)
# [[19 22]
#  [43 50]]

# Dot product of vectors
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])
print(np.dot(u, v))   # 32  ← 1*4 + 2*5 + 3*6

# Transpose
print(A.T)
# [[1 3]
#  [2 4]]

# Determinant
print(np.linalg.det(A))  # -2.0

# Inverse
print(np.linalg.inv(A))
# [[-2.   1. ]
#  [ 1.5 -0.5]]

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)
print(eigenvalues)   # [-0.372  5.372]
print(eigenvectors)

# Solve linear system Ax = b
b = np.array([5, 11])
x = np.linalg.solve(A, b)
print(x)  # [1. 2.]  ← solution to the system
```

---

## Comparison and Logic

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([3, 2, 5, 1, 5])

# Element-wise comparisons (return boolean arrays)
print(a == b)   # [False  True False False  True]
print(a != b)   # [ True False  True  True False]
print(a < b)    # [ True False  True False False]
print(a >= b)   # [False  True False  True  True]

# Logical operations on boolean arrays
x = np.array([True, True, False, False])
y = np.array([True, False, True, False])
print(np.logical_and(x, y))  # [ True False False False]
print(np.logical_or(x, y))   # [ True  True  True False]
print(np.logical_not(x))     # [False False  True  True]
print(np.logical_xor(x, y))  # [False  True  True False]

# Array-level logic (returns single bool)
arr = np.array([True, True, True])
print(np.all(arr))   # True  ← are ALL elements True?
print(np.any(arr))   # True  ← is ANY element True?

arr2 = np.array([True, False, True])
print(np.all(arr2))  # False
print(np.any(arr2))  # True

# Practical: are all students passing?
scores = np.array([85, 72, 91, 63, 88])
all_passing = np.all(scores >= 60)
some_failing = np.any(scores < 70)
print(f"All passing: {all_passing}")  # False
print(f"Some struggling: {some_failing}")  # True
```

---

## Performance Benchmarks

Let's see the speed difference concretely:

```python
import numpy as np
import time

n = 1_000_000  # One million elements

# Create data
data_list = list(range(n))
data_array = np.arange(n, dtype=float)

# ── Test 1: Squaring all elements ──
start = time.perf_counter()
result_list = [x**2 for x in data_list]
list_time = time.perf_counter() - start

start = time.perf_counter()
result_array = data_array ** 2
numpy_time = time.perf_counter() - start

print(f"List (loop):  {list_time:.4f}s")
print(f"NumPy (vect): {numpy_time:.4f}s")
print(f"Speedup: {list_time/numpy_time:.0f}x")

# ── Test 2: Computing sine ──
import math

start = time.perf_counter()
result_list = [math.sin(x) for x in data_list]
list_time = time.perf_counter() - start

start = time.perf_counter()
result_array = np.sin(data_array)
numpy_time = time.perf_counter() - start

print(f"\nSine List:  {list_time:.4f}s")
print(f"Sine NumPy: {numpy_time:.4f}s")
print(f"Speedup: {list_time/numpy_time:.0f}x")

# Typical results:
# Squaring: ~100x faster
# Sine:     ~200x faster
```

### When NOT to Use Vectorization

```python
# Sometimes loops are fine or necessary:
# 1. Very small arrays (overhead isn't worth it for < 10 elements)
# 2. Iterative algorithms where each step depends on the previous
# 3. Complex per-element logic that's hard to vectorize

# For iterative algorithms, consider Numba (JIT compiler)
# from numba import jit
# @jit(nopython=True)
# def my_iterative_function(arr): ...
```

---

## Real-World Vectorization Examples

### Example 1: Feature Normalization (Min-Max Scaling)

```python
# Normalize a feature to [0, 1] range
data = np.array([20, 35, 10, 50, 45, 15, 30])

# ❌ Loop version
result_loop = []
min_val = min(data)
max_val = max(data)
for x in data:
    result_loop.append((x - min_val) / (max_val - min_val))

# ✅ Vectorized version (much cleaner!)
min_val = data.min()
max_val = data.max()
normalized = (data - min_val) / (max_val - min_val)

print(normalized.round(3))
# [0.25  0.625 0.    1.    0.875 0.125 0.5  ]
```

### Example 2: Euclidean Distance

```python
# Distance between two points in N-dimensional space
point_a = np.array([1, 2, 3])
point_b = np.array([4, 6, 3])

# ❌ Loop
distance_loop = 0
for a, b in zip(point_a, point_b):
    distance_loop += (a - b) ** 2
distance_loop = distance_loop ** 0.5

# ✅ Vectorized
distance = np.sqrt(np.sum((point_a - point_b)**2))
# Or even cleaner:
distance = np.linalg.norm(point_a - point_b)

print(distance)  # 5.0
```

### Example 3: Portfolio Returns

```python
# Calculate daily returns for a stock portfolio
prices = np.array([100.0, 103.5, 101.2, 107.8, 105.3, 110.0])

# Daily percentage return = (today - yesterday) / yesterday * 100
# ❌ Loop
returns_loop = []
for i in range(1, len(prices)):
    returns_loop.append((prices[i] - prices[i-1]) / prices[i-1] * 100)

# ✅ Vectorized
returns = np.diff(prices) / prices[:-1] * 100
print(returns.round(2))
# [ 3.5  -2.22  6.52 -2.32  4.46]
```

---

## Summary

> [!success] Key Takeaways
>
> 1. **Vectorization** = replace loops with array operations → 100-1000x faster
> 2. **Arithmetic operators** (+, -, *, /, **, %) work element-wise on arrays
> 3. **ufuncs** (np.sin, np.exp, np.sqrt, etc.) apply to whole arrays at C speed
> 4. **Aggregation functions** (sum, mean, max, std) reduce arrays to scalars
> 5. **axis parameter** controls which dimension gets collapsed in aggregation
>    - `axis=0` = along rows (result per column)
>    - `axis=1` = along columns (result per row)
> 6. **`*` ≠ matrix multiply** — use `@` or `np.dot()` for that
> 7. `np.linalg` handles all linear algebra (determinant, inverse, solve, etc.)

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[03-indexing-and-slicing]] | [[05-broadcasting]] |

---

*Tags: #numpy #vectorization #ufuncs #element-wise #aggregation #axis #linear-algebra #performance*