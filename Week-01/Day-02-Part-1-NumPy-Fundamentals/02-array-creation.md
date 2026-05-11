# 02 — Array Creation
## Every Way to Build a NumPy Array

> [!quote] "Knowing how to create arrays efficiently is step one of writing good NumPy code."

---

## 🧭 Table of Contents

- [[#From Python Data]]
- [[#Filled Arrays]]
- [[#Range-Based Arrays]]
- [[#Random Arrays]]
- [[#Special Arrays]]
- [[#Loading Arrays from Files]]
- [[#Array Properties Recap]]
- [[#Choosing the Right Creation Function]]
- [[#Summary]]

---

## From Python Data

### `np.array()` — The Universal Constructor

The most flexible way to create a NumPy array — pass any Python list, tuple, or nested structure.

```python
import numpy as np

# ── From a list ──
arr1d = np.array([10, 20, 30, 40, 50])
print(arr1d)        # [10 20 30 40 50]
print(arr1d.dtype)  # int64

# ── From a list of floats ──
arr_float = np.array([1.5, 2.7, 3.14])
print(arr_float.dtype)  # float64

# ── From a tuple ──
arr_tuple = np.array((100, 200, 300))
print(arr_tuple)  # [100 200 300]

# ── 2D array from nested lists ──
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print(matrix)
# [[1 2 3]
#  [4 5 6]
#  [7 8 9]]
print(matrix.shape)  # (3, 3)

# ── 3D array ──
tensor = np.array([[[1, 2], [3, 4]],
                   [[5, 6], [7, 8]]])
print(tensor.shape)  # (2, 2, 2)

# ── Specifying dtype explicitly ──
arr_int32 = np.array([1, 2, 3], dtype=np.float32)
print(arr_int32)       # [1. 2. 3.]
print(arr_int32.dtype) # float32
```

> [!warning] Common Mistake: Unequal Row Lengths
> ```python
> # ❌ This creates an array of Python objects, NOT a proper 2D array!
> bad = np.array([[1, 2, 3], [4, 5]])  # Rows have different lengths!
> print(bad.dtype)  # object  ← Wrong!
> 
> # ✅ Always make sure all rows have the same length
> good = np.array([[1, 2, 3], [4, 5, 6]])  # ← Correct
> ```

---

## Filled Arrays

These functions create arrays pre-filled with a specific value — great for initializing arrays before filling them with real data.

### `np.zeros()` — Array of All Zeros

```python
# 1D array of zeros
z1 = np.zeros(5)
print(z1)        # [0. 0. 0. 0. 0.]
print(z1.dtype)  # float64  ← default is float!

# 2D array of zeros
z2 = np.zeros((3, 4))  # 3 rows, 4 columns
print(z2)
# [[0. 0. 0. 0.]
#  [0. 0. 0. 0.]
#  [0. 0. 0. 0.]]

# 3D array of zeros
z3 = np.zeros((2, 3, 4))  # 2 layers, 3 rows, 4 cols
print(z3.shape)  # (2, 3, 4)

# Zeros as integers
z_int = np.zeros((2, 3), dtype=np.int32)
print(z_int)
# [[0 0 0]
#  [0 0 0]]
```

### `np.ones()` — Array of All Ones

```python
# 1D
o1 = np.ones(4)
print(o1)  # [1. 1. 1. 1.]

# 2D
o2 = np.ones((2, 5))
print(o2)
# [[1. 1. 1. 1. 1.]
#  [1. 1. 1. 1. 1.]]

# Useful trick: multiply ones to get any constant
fives = np.ones((3, 3)) * 5
print(fives)
# [[5. 5. 5.]
#  [5. 5. 5.]
#  [5. 5. 5.]]
```

### `np.full()` — Array Filled with Any Value

```python
# Fill with 7
f = np.full((3, 3), 7)
print(f)
# [[7 7 7]
#  [7 7 7]
#  [7 7 7]]

# Fill with a float
f2 = np.full((2, 4), 3.14)
print(f2)
# [[3.14 3.14 3.14 3.14]
#  [3.14 3.14 3.14 3.14]]

# Fill with True (boolean array)
f3 = np.full((2, 3), True, dtype=bool)
print(f3)
# [[ True  True  True]
#  [ True  True  True]]
```

### `np.zeros_like()` and `np.ones_like()` — Match Another Array's Shape

```python
original = np.array([[1, 2, 3],
                     [4, 5, 6]])

# Create zeros with same shape AND dtype
z_like = np.zeros_like(original)
print(z_like)
# [[0 0 0]
#  [0 0 0]]
print(z_like.dtype)  # int64 (matches original)

o_like = np.ones_like(original, dtype=float)
print(o_like)
# [[1. 1. 1.]
#  [1. 1. 1.]]
```

> [!tip] When to Use These?
> - `zeros` → Initialize weight matrices before training ML models
> - `ones` → Create masks, or multiply to create constant arrays
> - `full` → Pre-fill with a sentinel value (like -1 for "no data")
> - `zeros_like` → Create an output array with same shape as input

---

## Range-Based Arrays

### `np.arange()` — Like Python's `range()`, but Returns an Array

```python
# Basic: just stop value
arr = np.arange(10)
print(arr)  # [0 1 2 3 4 5 6 7 8 9]

# start, stop
arr = np.arange(5, 15)
print(arr)  # [ 5  6  7  8  9 10 11 12 13 14]

# start, stop, step
arr = np.arange(0, 20, 2)
print(arr)  # [ 0  2  4  6  8 10 12 14 16 18]

# Floating point steps
arr = np.arange(0.0, 1.0, 0.1)
print(arr)  # [0.  0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9]

# Reverse (negative step)
arr = np.arange(10, 0, -1)
print(arr)  # [10  9  8  7  6  5  4  3  2  1]
```

> [!warning] Floating Point Gotcha with `arange`
> ```python
> # This can produce unexpected results due to floating point precision
> arr = np.arange(0, 1, 0.3)
> print(arr)  # [0.  0.3 0.6 0.9]  ← OK here
> 
> # But sometimes you get extra or missing elements
> # Use np.linspace() for float ranges instead!
> ```

### `np.linspace()` — Evenly Spaced Values (You Control the Count)

```python
# 5 evenly spaced values from 0 to 1 (inclusive)
arr = np.linspace(0, 1, 5)
print(arr)  # [0.   0.25 0.5  0.75 1.  ]

# 10 evenly spaced from 0 to 100
arr = np.linspace(0, 100, 10)
print(arr)  # [  0.  11.1 22.2 33.3 ... 100.]

# Exclude the endpoint
arr = np.linspace(0, 1, 5, endpoint=False)
print(arr)  # [0.  0.2 0.4 0.6 0.8]

# Also get the step size
arr, step = np.linspace(0, 10, 6, retstep=True)
print(arr)   # [ 0.  2.  4.  6.  8. 10.]
print(step)  # 2.0
```

### `arange` vs `linspace` — Which to Use?

| Situation | Use |
|-----------|-----|
| You know the **step size** | `np.arange(start, stop, step)` |
| You know the **number of points** | `np.linspace(start, stop, num)` |
| Working with **floats** | Prefer `np.linspace` (more reliable) |
| Creating integer sequences | `np.arange` |

```python
# arange: "give me steps of 0.1"
np.arange(0, 1, 0.1)     # → 10 elements (maybe 11 due to float issues!)

# linspace: "give me exactly 11 points"
np.linspace(0, 1, 11)    # → exactly 11 elements, guaranteed
```

---

## Random Arrays

Random arrays are everywhere in data science: weight initialization, shuffling data, simulations, tests.

```python
# Set a seed for reproducibility
np.random.seed(42)  # or use the newer Generator API

# ── Uniform distribution [0, 1) ──
arr = np.random.random((3, 3))  # 3x3 matrix of floats between 0 and 1
print(arr)
# [[0.374 0.951 0.732]
#  [0.599 0.156 0.058]
#  [0.866 0.708 0.021]]

# ── Uniform distribution [low, high) ──
arr = np.random.uniform(low=10, high=20, size=(2, 4))
print(arr)  # Values between 10 and 20

# ── Random integers ──
arr = np.random.randint(1, 100, size=(3, 3))  # Integers from 1 to 99
print(arr)
# [[52 93 15]
#  [72 61 21]
#  [83 87 75]]

# ── Normal distribution (mean=0, std=1) ──
arr = np.random.randn(4, 4)  # Standard normal
print(arr)

# Normal with custom mean and std
arr = np.random.normal(loc=100, scale=15, size=1000)  # IQ scores!
print(f"Mean: {arr.mean():.1f}, Std: {arr.std():.1f}")
# Mean: ~100.0, Std: ~15.0

# ── Random choice from an array ──
choices = np.random.choice([10, 20, 30, 40, 50], size=6)
print(choices)  # Random picks from the list

# Random choice without replacement (no repeats)
choices = np.random.choice(range(10), size=5, replace=False)
print(choices)  # 5 unique numbers from 0-9

# ── Shuffle an array in-place ──
arr = np.arange(10)
np.random.shuffle(arr)
print(arr)  # [3 1 7 4 0 9 2 6 5 8] (shuffled)
```

### Modern API: `np.random.default_rng()`

```python
# The modern, recommended way to use random in NumPy (since 1.17)
rng = np.random.default_rng(seed=42)

# Same operations, cleaner API
arr = rng.random((3, 3))
arr = rng.integers(0, 100, size=(3, 3))
arr = rng.standard_normal((4, 4))
arr = rng.choice([1, 2, 3, 4, 5], size=10)
```

> [!tip] Always Set a Seed!
> When writing code others will run, always set `np.random.seed(42)` (or use `rng = np.random.default_rng(42)`). This ensures your results are **reproducible**.

---

## Special Arrays

### `np.eye()` — Identity Matrix

```python
# 3x3 identity matrix (1s on diagonal, 0s elsewhere)
I = np.eye(3)
print(I)
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]

# 4x3 (not square)
I2 = np.eye(4, 3)
print(I2)
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]
#  [0. 0. 0.]]

# k parameter shifts the diagonal
I3 = np.eye(4, k=1)   # Diagonal shifted up by 1
print(I3)
# [[0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]
#  [0. 0. 0. 0.]]
```

### `np.diag()` — Diagonal Arrays

```python
# Create a diagonal matrix from a 1D array
d = np.diag([1, 2, 3, 4])
print(d)
# [[1 0 0 0]
#  [0 2 0 0]
#  [0 0 3 0]
#  [0 0 0 4]]

# Extract the diagonal FROM a 2D array
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print(np.diag(matrix))  # [1 5 9]  ← main diagonal elements
```

### `np.empty()` — Uninitialized Array (Fastest)

```python
# Creates an array WITHOUT initializing values
# Values are whatever was in memory at that location!
e = np.empty((3, 3))
print(e)  # Some garbage values — DO NOT USE before filling!

# Use when: you'll immediately fill every element anyway
# It's slightly faster than zeros() since it skips initialization
result = np.empty(1000)
for i in range(1000):
    result[i] = i ** 2  # Fill immediately
```

> [!danger] Never Read from `np.empty` Without Filling First!
> The values in an empty array are **garbage** — whatever happened to be in that memory location. Always fill before reading.

---

## Reshaping and Restructuring

### `reshape()` — Change the Shape

```python
# 1D → 2D
arr = np.arange(12)
print(arr)  # [ 0  1  2  3  4  5  6  7  8  9 10 11]

matrix = arr.reshape(3, 4)
print(matrix)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Using -1 to auto-calculate one dimension
matrix = arr.reshape(3, -1)   # NumPy figures out 4
matrix = arr.reshape(-1, 4)   # NumPy figures out 3

# 1D → 3D
tensor = arr.reshape(2, 2, 3)
print(tensor.shape)  # (2, 2, 3)

# 2D → 1D (flatten)
flat = matrix.flatten()  # Returns a COPY
flat2 = matrix.ravel()   # Returns a VIEW (faster, no copy)
```

### `np.concatenate()` — Join Arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Join 1D arrays
combined = np.concatenate([a, b])
print(combined)  # [1 2 3 4 5 6]

# Stack 2D arrays vertically (add rows)
A = np.ones((2, 3))
B = np.zeros((3, 3))
stacked = np.vstack([A, B])
print(stacked.shape)  # (5, 3)

# Stack 2D arrays horizontally (add columns)
A = np.ones((3, 2))
B = np.zeros((3, 4))
stacked = np.hstack([A, B])
print(stacked.shape)  # (3, 6)
```

---

## Loading Arrays from Files

```python
# Save array to file
arr = np.array([[1, 2, 3], [4, 5, 6]])
np.save('my_array.npy', arr)       # Binary format
np.savetxt('my_array.csv', arr, delimiter=',')  # Text/CSV

# Load array from file
loaded = np.load('my_array.npy')
loaded_txt = np.loadtxt('my_array.csv', delimiter=',')

# Save/load multiple arrays
np.savez('arrays.npz', a=arr, b=arr*2)
data = np.load('arrays.npz')
print(data['a'])  # Access by name
print(data['b'])
```

---

## Choosing the Right Creation Function

```
What do you need?
│
├── From existing data?          → np.array(list)
├── All zeros?                   → np.zeros(shape)
├── All ones?                    → np.ones(shape)
├── Any constant value?          → np.full(shape, value)
├── Like another array's shape?  → np.zeros_like(arr)
├── Integer sequence?            → np.arange(start, stop, step)
├── Evenly spaced floats?        → np.linspace(start, stop, num)
├── Random floats [0,1)?         → np.random.random(shape)
├── Random integers?             → np.random.randint(low, high, size)
├── Normal distribution?         → np.random.randn(shape)
├── Identity matrix?             → np.eye(n)
├── Diagonal matrix?             → np.diag(values)
└── Uninitialized (you'll fill)? → np.empty(shape)
```

---

## Summary

> [!success] Key Takeaways
>
> 1. `np.array()` converts Python lists/tuples to arrays — most flexible
> 2. `np.zeros()`, `np.ones()`, `np.full()` — create constant-filled arrays
> 3. `np.arange()` = range for arrays; `np.linspace()` = precise float spacing
> 4. `np.random.*` — tons of random distributions for simulation and ML
> 5. `np.eye()` and `np.diag()` — matrix algebra helpers
> 6. `reshape()` is your friend — change shape without changing data
> 7. **Always set `dtype` explicitly** when precision matters

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[01-introduction-to-numpy]] | [[03-indexing-and-slicing]] |

---

*Tags: #numpy #array-creation #zeros #ones #arange #linspace #random #reshape*