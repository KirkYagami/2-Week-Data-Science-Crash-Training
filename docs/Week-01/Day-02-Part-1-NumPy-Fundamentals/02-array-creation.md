# Array Creation
## Building NumPy Arrays the Right Way

You will create arrays hundreds of times in a single notebook session. Using the right creation function for the situation is not just style — it prevents bugs. Reaching for `np.arange` when you need `np.linspace` can give you a different number of elements than you expected, silently. Knowing the whole toolkit also means you write less code.

## Learning Objectives

- Create arrays from Python data, constant fills, ranges, random distributions, and special structures
- Choose between `np.arange` and `np.linspace` based on what you actually know: step size or count
- Set and convert dtypes deliberately, not by accident
- Use `np.random` for reproducible stochastic experiments
- Reshape and concatenate arrays without copying data unnecessarily

---

## From Python Data

### `np.array()` — The Universal Constructor

Pass any Python sequence: list, tuple, or nested structure. NumPy infers the dtype unless you override it.

```python
import numpy as np

# From a flat list
arr = np.array([10, 20, 30, 40, 50])
print(arr)        # Output: [10 20 30 40 50]
print(arr.dtype)  # Output: int64

# Floats stay floats
arr_f = np.array([1.5, 2.7, 3.14])
print(arr_f.dtype)  # Output: float64

# From a tuple — behaves identically
arr_t = np.array((100, 200, 300))
print(arr_t)  # Output: [100 200 300]

# 2-D from nested lists
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print(matrix)
# Output:
# [[1 2 3]
#  [4 5 6]
#  [7 8 9]]
print(matrix.shape)  # Output: (3, 3)

# 3-D tensor
tensor = np.array([[[1, 2], [3, 4]],
                   [[5, 6], [7, 8]]])
print(tensor.shape)  # Output: (2, 2, 2)

# Override the inferred dtype
arr_f32 = np.array([1, 2, 3], dtype=np.float32)
print(arr_f32)        # Output: [1. 2. 3.]
print(arr_f32.dtype)  # Output: float32
```

> [!warning] Ragged Rows Create Object Arrays
> ```python
> # Each row has a different length — NumPy cannot make a proper 2-D array
> bad = np.array([[1, 2, 3], [4, 5]])
> print(bad.dtype)   # Output: object
> print(bad.shape)   # Output: (2,)  ← 1-D array of Python lists!
>
> # Arithmetic on this will be slow and confusing.
> # Always make sure all rows have equal length.
> good = np.array([[1, 2, 3], [4, 5, 6]])
> print(good.shape)  # Output: (3, 3) ← proper 2-D array
> ```

---

## Filled Arrays

Use these when you need an array of a known shape before you fill it with real values — initializing weight matrices, building output buffers, creating masks.

### `np.zeros()` and `np.ones()`

```python
# 1-D
z = np.zeros(5)
print(z)        # Output: [0. 0. 0. 0. 0.]
print(z.dtype)  # Output: float64  ← default is float, not int

# 2-D
z2 = np.zeros((3, 4))   # 3 rows, 4 columns
print(z2.shape)  # Output: (3, 4)

# Integer zeros
z_int = np.zeros((2, 3), dtype=np.int32)
print(z_int)
# Output:
# [[0 0 0]
#  [0 0 0]]

# Ones
o = np.ones((2, 5))
print(o)
# Output:
# [[1. 1. 1. 1. 1.]
#  [1. 1. 1. 1. 1.]]
```

### `np.full()` — Any Constant Value

```python
# Fill with a specific value directly — cleaner than ones * value
f = np.full((3, 3), 7)
print(f)
# Output:
# [[7 7 7]
#  [7 7 7]
#  [7 7 7]]

# Useful for sentinel values
missing = np.full((4, 5), -1, dtype=np.int32)
# Use -1 to indicate "no data" before filling in real values

# Boolean array
mask = np.full((2, 3), True, dtype=bool)
print(mask)
# Output:
# [[ True  True  True]
#  [ True  True  True]]
```

### `np.zeros_like()` and `np.ones_like()`

These create a new array with the same shape and dtype as an existing array. Useful when you are building an output buffer that needs to match an input.

```python
data = np.array([[3.5, 1.2, 0.7],
                 [2.1, 4.8, 3.3]])

output = np.zeros_like(data)
print(output.dtype)  # Output: float64 — matches data
print(output.shape)  # Output: (2, 3)

# Override dtype if needed
counts = np.zeros_like(data, dtype=np.int32)
print(counts.dtype)  # Output: int32
```

> [!tip] When to Use Each
> - `zeros` — weight matrices before training, accumulator arrays before a sum
> - `ones` — masks where everything is initially included, or multiplicative identity
> - `full` — sentinel arrays, padding arrays before concatenation
> - `zeros_like` / `ones_like` — output arrays that must match input shape exactly

---

## Range-Based Arrays

### `np.arange()` — You Know the Step Size

```python
# arange(stop) — from 0 to stop-1
arr = np.arange(10)
print(arr)  # Output: [0 1 2 3 4 5 6 7 8 9]

# arange(start, stop) — from start to stop-1
arr = np.arange(5, 15)
print(arr)  # Output: [ 5  6  7  8  9 10 11 12 13 14]

# arange(start, stop, step)
arr = np.arange(0, 20, 2)
print(arr)  # Output: [ 0  2  4  6  8 10 12 14 16 18]

# Negative step (descending)
arr = np.arange(10, 0, -2)
print(arr)  # Output: [10  8  6  4  2]

# Float steps work, but read the warning below
arr = np.arange(0.0, 1.0, 0.2)
print(arr)  # Output: [0.  0.2 0.4 0.6 0.8]
```

> [!warning] Float Steps With `arange` Are Unreliable
> ```python
> # This should give 4 elements: 0.0, 0.3, 0.6, 0.9
> arr = np.arange(0, 1.2, 0.3)
> print(arr)  # Output: [0.  0.3 0.6 0.9 1.2]
> # Five elements! 0.3 + 0.3 + 0.3 + 0.3 does not equal exactly 1.2
> # due to floating point, so NumPy includes one extra value.
>
> # Use np.linspace when you work with floats.
> ```

### `np.linspace()` — You Know the Count

```python
# linspace(start, stop, num) — exactly num evenly spaced points
arr = np.linspace(0, 1, 5)
print(arr)  # Output: [0.   0.25 0.5  0.75 1.  ]
# Notice: endpoint is INCLUDED by default

# 11 points from 0 to 10 — useful for plotting
x = np.linspace(0, 10, 11)
print(x)  # Output: [ 0.  1.  2.  3.  4.  5.  6.  7.  8.  9. 10.]

# Exclude the endpoint
arr = np.linspace(0, 1, 5, endpoint=False)
print(arr)  # Output: [0.  0.2 0.4 0.6 0.8]

# Get the step size back
arr, step = np.linspace(0, 10, 6, retstep=True)
print(arr)   # Output: [ 0.  2.  4.  6.  8. 10.]
print(step)  # Output: 2.0
```

### `arange` vs `linspace` — Decision Rule

| Question | Use |
|----------|-----|
| "I want steps of size 0.5" | `np.arange(start, stop, 0.5)` |
| "I want exactly 100 points" | `np.linspace(start, stop, 100)` |
| "I want integer sequences" | `np.arange` |
| "I want float ranges reliably" | `np.linspace` |

```python
# Plotting a sine curve: you want exactly 200 points, not a specific step
x = np.linspace(0, 2 * np.pi, 200)
y = np.sin(x)

# Generating batch indices: you want steps of 32
batches = np.arange(0, 10000, 32)
```

---

## Random Arrays

Random arrays appear everywhere: weight initialization in neural networks, shuffling training data, Monte Carlo simulations, generating test data.

### Setting a Seed

```python
# Always set a seed when reproducibility matters
np.random.seed(42)
# or use the modern Generator API (recommended for new code):
rng = np.random.default_rng(seed=42)
```

> [!tip] Use `default_rng` for New Code
> `np.random.default_rng()` is the modern NumPy random API (since 1.17). It is statistically better (PCG64 algorithm) and avoids global state. The older `np.random.seed()` still works but is less safe in multi-threaded code.

### Common Distributions

```python
import numpy as np
rng = np.random.default_rng(seed=42)

# Uniform floats in [0, 1)
arr = rng.random((3, 4))
print(arr.shape)  # Output: (3, 4)
print(arr.min(), arr.max())  # Output: something between 0 and 1

# Uniform floats in a custom range [low, high)
arr = rng.uniform(low=10.0, high=20.0, size=(2, 5))
print(arr.round(2))
# Output: values between 10 and 20

# Random integers in [low, high) — high is exclusive
dice_rolls = rng.integers(low=1, high=7, size=10)
print(dice_rolls)
# Output: [5 2 6 1 3 4 6 3 2 5]  (actual values vary)

# Standard normal distribution (mean=0, std=1)
noise = rng.standard_normal((4, 4))
print(noise.round(3))
# Values centered around 0, most within -3 to +3

# Normal with custom mean and std
heights = rng.normal(loc=170, scale=10, size=1000)  # adult heights in cm
print(f"Mean: {heights.mean():.1f}, Std: {heights.std():.1f}")
# Output: Mean: ~170.0, Std: ~10.0

# Random choice from an array
categories = rng.choice(['A', 'B', 'C'], size=6, p=[0.5, 0.3, 0.2])
print(categories)
# Output: picks from A/B/C with specified probabilities

# Without replacement (no repeats)
sample = rng.choice(np.arange(10), size=5, replace=False)
print(sample)  # 5 unique values from 0-9

# Shuffle in-place
arr = np.arange(10)
rng.shuffle(arr)
print(arr)  # Output: randomized order
```

> [!warning] Reproducibility Requires a Fixed Seed
> Without a seed, every run of your code produces different results. This makes debugging and collaboration nearly impossible. Always set a seed in notebooks that others will run or review.

---

## Special Matrices

### `np.eye()` — Identity Matrix

The identity matrix is the matrix equivalent of the number 1: any matrix multiplied by the identity matrix returns itself. Essential in linear algebra and used in regularization.

```python
I = np.eye(3)
print(I)
# Output:
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]

# Rectangular identity (not square)
I2 = np.eye(4, 3)
print(I2)
# Output:
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]
#  [0. 0. 0.]]

# Offset diagonal with k parameter
I3 = np.eye(4, k=1)   # 1 above main diagonal
print(I3)
# Output:
# [[0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]
#  [0. 0. 0. 0.]]
```

### `np.diag()` — Build or Extract Diagonals

```python
# Build a diagonal matrix from a 1-D array
d = np.diag([10, 20, 30, 40])
print(d)
# Output:
# [[10  0  0  0]
#  [ 0 20  0  0]
#  [ 0  0 30  0]
#  [ 0  0  0 40]]

# Extract the diagonal FROM an existing 2-D array
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print(np.diag(matrix))  # Output: [1 5 9]

# Off-diagonal extraction
print(np.diag(matrix, k=1))   # Output: [2 6]  ← one above main
print(np.diag(matrix, k=-1))  # Output: [4 8]  ← one below main
```

### `np.empty()` — Skip Initialization for Speed

```python
# Creates an array WITHOUT zeroing the memory
# Values are garbage — whatever was previously at those addresses
buf = np.empty((4, 4))
print(buf)  # unpredictable values

# Correct use: allocate first, fill every element before reading
result = np.empty(1_000_000)
result[:] = np.arange(1_000_000) ** 2  # fill immediately
```

> [!warning] Never Read From `np.empty` Without Filling First
> The values in an empty array are not zero — they are memory garbage. Reading from an unfilled `empty` array produces unpredictable results that change every run. The only reason to use `empty` over `zeros` is the small speed gain from skipping initialization, which matters only for very large arrays you will fill immediately anyway.

---

## Reshaping and Combining

### `reshape()` — Change Shape, Keep Data

```python
arr = np.arange(12)
print(arr)  # Output: [ 0  1  2  3  4  5  6  7  8  9 10 11]

# Reshape to 2-D
m = arr.reshape(3, 4)
print(m)
# Output:
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Use -1 to let NumPy calculate one dimension
m2 = arr.reshape(3, -1)    # -1 → 4, because 12 / 3 = 4
m3 = arr.reshape(-1, 4)    # -1 → 3, because 12 / 4 = 3
t  = arr.reshape(2, 2, 3)  # 3-D tensor
print(t.shape)  # Output: (2, 2, 3)

# Flatten back to 1-D
flat = m.flatten()   # always returns a COPY
flat2 = m.ravel()    # returns a VIEW when possible (faster, less memory)
```

> [!warning] `reshape` Returns a View
> ```python
> arr = np.arange(6)
> m = arr.reshape(2, 3)
> m[0, 0] = 999
> print(arr)  # Output: [999   1   2   3   4   5]
> # The original array changed!
> # Use arr.reshape(2, 3).copy() if you need independence.
> ```

### Combining Arrays

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# Stack vertically (add rows)
v = np.vstack([a, b])
print(v.shape)  # Output: (4, 2)

# Stack horizontally (add columns)
h = np.hstack([a, b])
print(h.shape)  # Output: (2, 4)

# General concatenation along any axis
c0 = np.concatenate([a, b], axis=0)  # same as vstack
c1 = np.concatenate([a, b], axis=1)  # same as hstack
```

---

## Saving and Loading

```python
arr = np.array([[1.0, 2.0], [3.0, 4.0]])

# Binary format — fast, preserves dtype exactly
np.save('data.npy', arr)
loaded = np.load('data.npy')

# Multiple arrays in one file
np.savez('multi.npz', features=arr, labels=np.array([0, 1]))
data = np.load('multi.npz')
print(data['features'])
print(data['labels'])

# Text / CSV format — human-readable but slower
np.savetxt('data.csv', arr, delimiter=',', fmt='%.4f')
loaded_txt = np.loadtxt('data.csv', delimiter=',')
```

---

## Choosing the Right Creation Function

```
What do you need?
│
├── From existing Python data?       → np.array(data)
├── All zeros?                       → np.zeros(shape)
├── All ones?                        → np.ones(shape)
├── A specific constant?             → np.full(shape, value)
├── Same shape as existing array?    → np.zeros_like(arr)
├── Integer or float sequence?
│     ├── Know the step size         → np.arange(start, stop, step)
│     └── Know the count             → np.linspace(start, stop, num)
├── Random uniform floats [0, 1)?    → rng.random(shape)
├── Random integers?                 → rng.integers(low, high, size)
├── Normal distribution?             → rng.standard_normal(shape)
├── Random picks from array?         → rng.choice(arr, size)
├── Identity matrix?                 → np.eye(n)
├── Diagonal matrix?                 → np.diag(values)
└── Uninitialized (fill immediately) → np.empty(shape)
```

---

> [!success] Key Takeaways
> - `np.array()` converts Python data; all other functions generate arrays directly
> - `np.zeros`, `np.ones`, `np.full` — pre-fill with constants; `zeros_like` matches an existing shape
> - `arange` when you know the step; `linspace` when you know the count, especially with floats
> - Always set a seed before random operations in code others will read or reproduce
> - `reshape` returns a view — modifying the result modifies the original
> - `flatten()` makes a copy; `ravel()` makes a view when possible

---

[[01-introduction-to-numpy]] | [[03-indexing-and-slicing]]
