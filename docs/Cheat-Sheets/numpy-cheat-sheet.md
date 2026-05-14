# NumPy Cheat Sheet

NumPy is the numerical foundation of the Python data science stack. Everything in pandas, scikit-learn, and PyTorch ultimately lives on top of NumPy arrays. Knowing how to manipulate arrays efficiently — without Python loops — is what separates fast, production-ready code from slow notebook experiments.

---

## Array Creation

### np.array — convert Python lists to arrays

Use when you have existing data in a list or nested list and need to move it into NumPy for vectorized operations.

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([[1, 2, 3], [4, 5, 6]])

print(a)       # Output: [1 2 3 4]
print(b)
# Output:
# [[1 2 3]
#  [4 5 6]]

# Force a specific dtype — important when feeding into ML models
c = np.array([1, 2, 3], dtype=np.float32)
print(c.dtype)  # Output: float32
```

### np.zeros, np.ones, np.full — placeholder arrays

Use to initialize weight matrices, mask arrays, or any structure you will fill in later. Faster and cleaner than list comprehensions.

```python
z = np.zeros((3, 4))          # 3x4 matrix of 0.0
o = np.ones((2, 3))           # 2x3 matrix of 1.0
f = np.full((3, 3), 7)        # 3x3 matrix filled with 7

print(z.shape)   # Output: (3, 4)
print(f)
# Output:
# [[7 7 7]
#  [7 7 7]
#  [7 7 7]]
```

### np.arange and np.linspace — sequential ranges

`np.arange` mirrors Python's `range` but returns an array. Use it for integer sequences.
`np.linspace` gives you a fixed number of evenly spaced floats between two endpoints — essential for plotting function curves or building grids.

```python
# arange: start, stop (exclusive), step
seq = np.arange(0, 10, 2)
print(seq)   # Output: [0 2 4 6 8]

# linspace: start, stop (inclusive), num points
grid = np.linspace(0, 1, 5)
print(grid)  # Output: [0.   0.25 0.5  0.75 1.  ]
```

### np.eye — identity matrix

Use to initialize weight matrices in linear algebra, or to one-hot encode class labels manually.

```python
eye = np.eye(4)
print(eye)
# Output:
# [[1. 0. 0. 0.]
#  [0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]]
```

### np.random — reproducible random arrays

Always set a seed before generating random data in experiments. Without it, results change every run and debugging becomes unreliable.

```python
np.random.seed(42)

rand_uniform = np.random.rand(3, 3)      # uniform [0, 1)
rand_normal  = np.random.randn(3, 3)     # standard normal, mean=0, std=1
rand_int     = np.random.randint(0, 10, size=(2, 4))  # integers in [0, 10)

print(rand_int)
# Output (with seed 42):
# [[6 3 7 4]
#  [6 9 2 6]]
```

---

## Array Attributes

### .shape, .ndim, .size — understand the structure before operating on it

Always inspect these when loading data from an external source. Mismatched shapes are the single most common source of NumPy errors.

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

print(arr.shape)   # Output: (2, 3)   — 2 rows, 3 columns
print(arr.ndim)    # Output: 2        — number of dimensions
print(arr.size)    # Output: 6        — total number of elements
```

### .dtype — know your data type

Feeding float64 data into a model that expects float32 causes silent precision loss or errors. Check dtype early.

```python
a = np.array([1.0, 2.0, 3.0])
b = np.array([1, 2, 3])

print(a.dtype)   # Output: float64
print(b.dtype)   # Output: int64

# Cast to a different type
c = b.astype(np.float32)
print(c.dtype)   # Output: float32
```

### .T — transpose a 2D array

Transposing is needed constantly: aligning matrices for dot products, converting row vectors to column vectors, reshaping feature matrices.

```python
m = np.array([[1, 2, 3], [4, 5, 6]])
print(m.shape)     # Output: (2, 3)
print(m.T.shape)   # Output: (3, 2)
print(m.T)
# Output:
# [[1 4]
#  [2 5]
#  [3 6]]
```

---

## Indexing & Slicing

### 1D slicing

Standard Python slice syntax. The key difference from lists: NumPy slices return views, not copies. Modifying a slice modifies the original array.

```python
a = np.arange(10)   # [0 1 2 3 4 5 6 7 8 9]

print(a[3])         # Output: 3
print(a[2:6])       # Output: [2 3 4 5]
print(a[::-1])      # Output: [9 8 7 6 5 4 3 2 1 0]
print(a[::2])       # Output: [0 2 4 6 8]
```

### 2D indexing

Row first, column second. Use `:` to select entire rows or columns.

```python
m = np.array([[10, 20, 30],
              [40, 50, 60],
              [70, 80, 90]])

print(m[1, 2])      # Output: 60
print(m[0, :])      # Output: [10 20 30]  — first row
print(m[:, 1])      # Output: [20 50 80]  — second column
print(m[0:2, 1:3])
# Output:
# [[20 30]
#  [50 60]]
```

### Boolean indexing — filter rows by condition

This is the NumPy equivalent of a SQL WHERE clause. It is used constantly in data cleaning and feature selection.

```python
scores = np.array([55, 72, 88, 43, 91, 67])

mask = scores > 70
print(mask)             # Output: [False  True  True False  True False]
print(scores[mask])     # Output: [72 88 91]

# Combine conditions
print(scores[(scores > 60) & (scores < 90)])  # Output: [72 88 67]
```

### Fancy indexing — select arbitrary rows or elements

Use when you need a specific non-contiguous subset of rows — common when shuffling datasets or picking top-k samples.

```python
arr = np.array([10, 20, 30, 40, 50])

indices = [0, 2, 4]
print(arr[indices])   # Output: [10 30 50]

m = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])
print(m[[0, 2, 3]])
# Output:
# [[1 2]
#  [5 6]
#  [7 8]]
```

### np.where — conditional element selection

Use to replace values based on a condition without writing a loop. Common for clamping outliers or creating binary label arrays.

```python
a = np.array([5, -3, 8, -1, 0, 4])

# np.where(condition, value_if_true, value_if_false)
result = np.where(a > 0, a, 0)
print(result)   # Output: [5 0 8 0 0 4]

# Get indices where condition holds
idx = np.where(a < 0)
print(idx)      # Output: (array([1, 3]),)
```

---

## Reshaping

### .reshape() — change dimensions without copying data

Use before feeding data into a model that expects a specific input shape. Reshape is view-based — it does not copy data as long as the array is contiguous.

```python
a = np.arange(12)
print(a.shape)           # Output: (12,)

b = a.reshape(3, 4)
print(b.shape)           # Output: (3, 4)

# Use -1 to let NumPy infer one dimension
c = a.reshape(2, -1)
print(c.shape)           # Output: (2, 6)
```

### .flatten() vs .ravel() — collapse to 1D

`.flatten()` always returns a copy. `.ravel()` returns a view when possible (faster, no memory overhead). Use `.ravel()` in performance-sensitive code.

```python
m = np.array([[1, 2, 3], [4, 5, 6]])

flat = m.flatten()   # copy
rav  = m.ravel()     # view (usually)

print(flat)   # Output: [1 2 3 4 5 6]
print(rav)    # Output: [1 2 3 4 5 6]

# Modifying rav changes m; modifying flat does not
rav[0] = 99
print(m[0, 0])    # Output: 99
```

### np.newaxis and np.expand_dims — add a dimension

Broadcasting requires arrays to have compatible shapes. Adding a dimension converts a 1D array into a row or column vector, enabling operations between arrays of different ranks.

```python
a = np.array([1, 2, 3])   # shape (3,)

row = a[np.newaxis, :]       # shape (1, 3)
col = a[:, np.newaxis]       # shape (3, 1)

print(row.shape)   # Output: (1, 3)
print(col.shape)   # Output: (3, 1)

# Equivalent with expand_dims
col2 = np.expand_dims(a, axis=1)
print(col2.shape)  # Output: (3, 1)
```

### np.squeeze — remove size-1 dimensions

Model outputs often come back as shape `(batch, 1)` when you need `(batch,)`. Squeeze removes those extra dimensions.

```python
a = np.array([[[1], [2], [3]]])   # shape (1, 3, 1)
print(a.shape)                     # Output: (1, 3, 1)

b = np.squeeze(a)
print(b.shape)                     # Output: (3,)

# Squeeze only a specific axis
c = np.squeeze(a, axis=0)
print(c.shape)                     # Output: (3, 1)
```

---

## Math Operations

### Element-wise arithmetic

NumPy operations apply element-by-element without loops. This is orders of magnitude faster than equivalent Python for-loops on large arrays.

```python
a = np.array([1.0, 2.0, 3.0, 4.0])
b = np.array([10.0, 20.0, 30.0, 40.0])

print(a + b)    # Output: [11. 22. 33. 44.]
print(b / a)    # Output: [10. 10. 10. 10.]
print(a ** 2)   # Output: [ 1.  4.  9. 16.]
print(a * b)    # Output: [ 10.  40.  90. 160.]
```

### np.sqrt, np.log, np.exp — universal functions (ufuncs)

These operate element-wise and are implemented in C — always prefer them over math.sqrt in a loop. Common in feature engineering (log-transform skewed features, normalize with exp).

```python
a = np.array([1.0, 4.0, 9.0, 16.0])

print(np.sqrt(a))   # Output: [1. 2. 3. 4.]
print(np.log(a))    # Output: [0.     1.386  2.197  2.773]
print(np.exp(a))    # Output: [2.718e+00 5.460e+01 8.103e+03 8.887e+06]
```

### np.abs, np.round — cleaning and rounding

Use `np.abs` when computing distances, residuals, or errors. Use `np.round` when preparing outputs for display or when floating point precision is causing comparison issues.

```python
a = np.array([-3.7, 1.2, -0.5, 4.9])

print(np.abs(a))          # Output: [3.7 1.2 0.5 4.9]
print(np.round(a))        # Output: [-4.  1. -0.  5.]
print(np.round(a, 1))     # Output: [-3.7  1.2 -0.5  4.9]
```

### np.clip — bound values to a range

Use to cap model predictions, prevent log(0), or sanitize user input arrays. More readable than a combined where expression.

```python
a = np.array([-5, 0, 3, 7, 12, 20])

clipped = np.clip(a, 0, 10)
print(clipped)   # Output: [ 0  0  3  7 10 10]

# Common use: prevent log(0) by clipping probabilities
probs = np.array([0.0, 0.3, 0.7, 1.0])
safe  = np.clip(probs, 1e-7, 1 - 1e-7)
```

---

## Aggregations

### np.sum, np.mean, np.std, np.var

The `axis` parameter is critical. Axis 0 aggregates across rows (collapses rows). Axis 1 aggregates across columns (collapses columns). Getting this wrong is one of the most common bugs in ML preprocessing.

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

print(np.sum(m))           # Output: 21         — all elements
print(np.sum(m, axis=0))   # Output: [5 7 9]    — sum each column
print(np.sum(m, axis=1))   # Output: [ 6 15]    — sum each row

print(np.mean(m, axis=0))  # Output: [2.5 3.5 4.5]
print(np.std(m, axis=1))   # Output: [0.816 0.816]
```

### np.min, np.max — global and per-axis extremes

```python
m = np.array([[3, 1, 4], [1, 5, 9], [2, 6, 5]])

print(np.max(m))           # Output: 9
print(np.max(m, axis=0))   # Output: [3 6 9]   — max in each column
print(np.min(m, axis=1))   # Output: [1 1 2]   — min in each row
```

### np.argmin, np.argmax — index of the extreme value

Use when you need the position, not the value. Essential for finding the predicted class in a classification model (argmax over output probabilities).

```python
scores = np.array([0.1, 0.6, 0.2, 0.05, 0.05])

print(np.argmax(scores))   # Output: 1   — index of highest probability

m = np.array([[3, 1, 4], [1, 5, 9]])
print(np.argmax(m, axis=1))   # Output: [2 2]  — col index of max in each row
print(np.argmin(m, axis=0))   # Output: [1 0 0] — row index of min in each col
```

### np.cumsum — running totals

Use for cumulative distributions, running revenue, or building prefix sums for sliding window problems.

```python
a = np.array([1, 2, 3, 4, 5])
print(np.cumsum(a))   # Output: [ 1  3  6 10 15]

m = np.array([[1, 2], [3, 4]])
print(np.cumsum(m, axis=0))
# Output:
# [[1 2]
#  [4 6]]
```

---

## Broadcasting

### The rule

Broadcasting lets NumPy perform operations on arrays with different shapes without copying data. The rule: dimensions are compared trailing-to-leading. Two dimensions are compatible if they are equal, or if one of them is 1.

### Add a scalar to an array

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]])

print(a + 10)
# Output:
# [[11 12 13]
#  [14 15 16]]
```

### Add a row vector to every row of a matrix

The row vector has shape `(3,)` which broadcasts with `(2, 3)`.

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

bias = np.array([10, 20, 30])   # shape (3,)

result = m + bias
print(result)
# Output:
# [[11 22 33]
#  [14 25 36]]
```

### Add a column vector to every column of a matrix

The column vector must have shape `(n, 1)` to broadcast with `(n, m)`.

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

col = np.array([[100],
                [200]])   # shape (2, 1)

result = m + col
print(result)
# Output:
# [[101 102 103]
#  [204 205 206]]
```

### Normalize each column (zero-mean, unit variance)

Broadcasting makes this a one-liner — no loops needed.

```python
data = np.array([[2.0, 400.0],
                 [4.0, 800.0],
                 [6.0, 200.0]])

mean = data.mean(axis=0)   # shape (2,)
std  = data.std(axis=0)    # shape (2,)

normalized = (data - mean) / std
print(normalized.round(2))
# Output:
# [[-1.   -0.27]
#  [ 0.    1.07]
#  [ 1.   -0.8 ]]
```

---

## Linear Algebra

### np.dot and @ — matrix multiplication

Use `@` (Python 3.5+) for readability. `np.dot` is equivalent for 2D arrays. Matrix multiplication is the core of neural network forward passes and linear regression solutions.

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

print(A @ B)
# Output:
# [[19 22]
#  [43 50]]

# Same result:
print(np.dot(A, B))

# Dot product of two vectors (returns scalar)
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])
print(np.dot(u, v))   # Output: 32
```

### np.linalg.inv — matrix inverse

Use to solve linear systems analytically. In practice, prefer `np.linalg.solve` over computing the inverse explicitly — it is numerically more stable and faster.

```python
A = np.array([[2.0, 1.0], [5.0, 3.0]])
A_inv = np.linalg.inv(A)

print(A_inv)
# Output:
# [[ 3. -1.]
#  [-5.  2.]]

# Verify: A @ A_inv should be identity
print(np.round(A @ A_inv))
# Output:
# [[1. 0.]
#  [0. 1.]]
```

### np.linalg.solve — solve a linear system Ax = b

More numerically stable than computing the inverse. Use for ordinary least squares, solving simultaneous equations in feature engineering.

```python
A = np.array([[3.0, 1.0], [1.0, 2.0]])
b = np.array([9.0, 8.0])

x = np.linalg.solve(A, b)
print(x)   # Output: [2. 3.]

# Verify
print(A @ x)   # Output: [9. 8.]
```

### np.linalg.eig — eigenvalues and eigenvectors

Used in PCA, spectral clustering, and stability analysis. The eigenvectors of the covariance matrix define the principal components.

```python
A = np.array([[4.0, 1.0], [2.0, 3.0]])

eigenvalues, eigenvectors = np.linalg.eig(A)
print(eigenvalues)
# Output: [5. 2.]

print(eigenvectors)
# Output:
# [[0.707 -0.447]
#  [0.707  0.894]]
```

### np.linalg.norm — vector and matrix norms

Use to normalize feature vectors, compute distances, regularize weights, or check convergence.

```python
v = np.array([3.0, 4.0])

print(np.linalg.norm(v))           # Output: 5.0   — L2 norm (Euclidean)
print(np.linalg.norm(v, ord=1))    # Output: 7.0   — L1 norm (Manhattan)

# Normalize to unit vector
v_unit = v / np.linalg.norm(v)
print(v_unit)    # Output: [0.6 0.8]
```

---

## Sorting & Searching

### np.sort — return a sorted copy

Use when you need sorted values. Does not modify the original. Pass `axis` to sort along rows or columns.

```python
a = np.array([3, 1, 4, 1, 5, 9, 2, 6])

print(np.sort(a))         # Output: [1 1 2 3 4 5 6 9]
print(np.sort(a)[::-1])   # Output: [9 6 5 4 3 2 1 1]  — descending

m = np.array([[3, 1, 2], [6, 4, 5]])
print(np.sort(m, axis=1))
# Output:
# [[1 2 3]
#  [4 5 6]]
```

### np.argsort — indices that would sort the array

Use when you need the rank of each element, or when you want to reorder another array in the same order as this one. Essential for top-k retrieval.

```python
scores = np.array([0.3, 0.9, 0.1, 0.7])

order = np.argsort(scores)
print(order)           # Output: [2 0 3 1]  — ascending order of indices

# Top-2 highest scores
top2 = np.argsort(scores)[-2:][::-1]
print(top2)            # Output: [1 3]
print(scores[top2])    # Output: [0.9 0.7]
```

### np.unique — find unique values

Use to count distinct categories, detect duplicates, or get the vocabulary of a categorical feature.

```python
labels = np.array([2, 1, 3, 1, 2, 2, 3])

print(np.unique(labels))
# Output: [1 2 3]

# Get counts as well
values, counts = np.unique(labels, return_counts=True)
print(values)   # Output: [1 2 3]
print(counts)   # Output: [2 3 2]
```

### np.in1d — membership test

Use to check which elements of one array appear in another — equivalent to a SQL IN clause. Useful for filtering datasets by a list of allowed IDs.

```python
user_ids = np.array([101, 205, 307, 410, 512])
active   = np.array([205, 410, 512])

mask = np.in1d(user_ids, active)
print(mask)
# Output: [False  True False  True  True]

print(user_ids[mask])
# Output: [205 410 512]
```

---

## Stacking & Splitting

### np.concatenate — join arrays along an existing axis

The most general stacking function. All arrays must have the same shape except along the axis you are concatenating.

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# Stack vertically (axis=0 — add rows)
print(np.concatenate([a, b], axis=0))
# Output:
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

# Stack horizontally (axis=1 — add columns)
print(np.concatenate([a, b], axis=1))
# Output:
# [[1 2 5 6]
#  [3 4 7 8]]
```

### np.vstack and np.hstack — shorthand stacking

`np.vstack` stacks row-by-row (vertically). `np.hstack` stacks column-by-column (horizontally). More readable than specifying `axis` explicitly.

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

print(np.vstack([a, b]))
# Output:
# [[1 2 3]
#  [4 5 6]]

print(np.hstack([a, b]))
# Output: [1 2 3 4 5 6]

# Common pattern: combine feature matrix with labels
features = np.ones((3, 4))
labels   = np.zeros((3, 1))
dataset  = np.hstack([features, labels])
print(dataset.shape)   # Output: (3, 5)
```

### np.split — divide an array into sub-arrays

Use to split a dataset into train/validation/test sets, or to break batches into smaller chunks.

```python
a = np.arange(12)

# Split into 3 equal parts
parts = np.split(a, 3)
print(parts[0])   # Output: [0 1 2 3]
print(parts[1])   # Output: [4 5 6 7]
print(parts[2])   # Output: [8 9 10 11]

# Split at specific indices
chunks = np.split(a, [4, 9])
print(chunks[0])   # Output: [0 1 2 3]
print(chunks[1])   # Output: [4 5 6 7 8]
print(chunks[2])   # Output: [9 10 11]
```

---

## Saving & Loading

### np.save and np.load — binary NumPy format

Use `.npy` format when you need fast, lossless serialization of a single array. Preserves dtype exactly. Much faster to load than CSV.

```python
arr = np.array([[1.0, 2.0], [3.0, 4.0]])

np.save("my_array.npy", arr)
loaded = np.load("my_array.npy")

print(loaded)
# Output:
# [[1. 2.]
#  [3. 4.]]
print(loaded.dtype)   # Output: float64
```

### np.savez — save multiple arrays to one file

Use when you need to bundle several related arrays (e.g., features, labels, and metadata) into a single file.

```python
X = np.random.rand(100, 10)
y = np.random.randint(0, 2, 100)

np.savez("dataset.npz", features=X, labels=y)

data = np.load("dataset.npz")
print(data["features"].shape)   # Output: (100, 10)
print(data["labels"].shape)     # Output: (100,)
```

### np.savetxt and np.loadtxt — human-readable text files

Use when the output needs to be readable by other tools (Excel, R, etc.) or when you need to share data with people who cannot use NumPy. Slower and larger than binary formats.

```python
arr = np.array([[1.0, 2.5, 3.1],
                [4.2, 5.0, 6.7]])

np.savetxt("data.csv", arr, delimiter=",", fmt="%.2f", header="a,b,c")
loaded = np.loadtxt("data.csv", delimiter=",", skiprows=1)

print(loaded)
# Output:
# [[1.   2.5  3.1]
#  [4.2  5.   6.7]]
```

### np.genfromtxt — robust loading with missing values

Use instead of `np.loadtxt` when your file has missing values or inconsistent rows. It fills gaps with `np.nan` by default.

```python
# Assume data.csv contains:
# 1.0, 2.0, 3.0
# 4.0, , 6.0

data = np.genfromtxt("data.csv", delimiter=",", filling_values=0.0)
print(data)
# Output:
# [[1. 2. 3.]
#  [4. 0. 6.]]
```

> [!tip]
> For most real-world tabular data work, prefer `pandas.read_csv` over `np.loadtxt`. NumPy's text loaders are best for homogeneous numeric arrays with no headers or minimal structure.

> [!warning]
> NumPy slices return views, not copies. If you modify a slice, you modify the original array. Use `.copy()` explicitly when you need an independent array: `b = a[1:3].copy()`.

> [!warning]
> `axis=0` always means "collapse across rows" (operates column-by-column). `axis=1` means "collapse across columns" (operates row-by-row). Drawing a small 2x3 matrix and tracing the axis direction by hand is the fastest way to fix this in your mental model.
