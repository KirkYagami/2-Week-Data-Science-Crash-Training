# 05 — Broadcasting
## Operating on Arrays of Different Shapes

> [!quote] "Broadcasting is NumPy's way of making arithmetic between differently shaped arrays work as if by magic — but it follows strict, learnable rules."

---

## 🧭 Table of Contents

- [[#What is Broadcasting?]]
- [[#Broadcasting Rules]]
- [[#Simple Examples — Building Intuition]]
- [[#The Rules in Action]]
- [[#When Broadcasting Fails]]
- [[#Real-World Use Cases]]
- [[#Advanced Broadcasting]]
- [[#Summary]]

---

## What is Broadcasting?

**Broadcasting** is NumPy's mechanism for performing arithmetic between arrays of **different shapes**.

Without broadcasting, every operation would require arrays of exactly the same size. Broadcasting extends NumPy's power by automatically "stretching" smaller arrays to match larger ones — **without actually copying any data**.

### The Basic Idea

```python
import numpy as np

# Scalar + Array (the simplest broadcasting)
arr = np.array([1, 2, 3, 4, 5])
result = arr + 10

# Conceptually, NumPy treats it as:
# arr + [10, 10, 10, 10, 10]  ← scalar "broadcasted" to match arr's shape
# But it doesn't actually create that array — it's just conceptual!

print(result)  # [11 12 13 14 15]
```

### Why This Matters

Broadcasting lets you write clean, memory-efficient code:

```python
# Without broadcasting, you'd have to do this manually:
arr = np.array([1, 2, 3, 4, 5])
scalar_array = np.full_like(arr, 10)  # [10 10 10 10 10] — wastes memory!
result = arr + scalar_array

# With broadcasting, just:
result = arr + 10  # NumPy handles it — no extra array created!
```

---

## Broadcasting Rules

NumPy broadcasts two arrays following **3 strict rules**. Learn these and broadcasting will always make sense.

> [!important] The 3 Broadcasting Rules
>
> **Rule 1:** If the two arrays have different numbers of dimensions, the **shape of the one with fewer dimensions is padded with 1s on the LEFT**.
>
> **Rule 2:** Arrays with size **1 along a particular dimension** act as if they have the size of the array with the largest size in that dimension.
>
> **Rule 3:** If arrays have different sizes in a dimension and **neither is 1**, a **ValueError** is raised — broadcasting fails.

### Applying the Rules — Step by Step

```
Example: shape (3,) + shape (3, 3)

Step 1 (Rule 1): Pad the 1D array with 1s on the left
  (3,) → (1, 3)

Step 2 (Rule 2): Stretch the 1 dimension to match
  (1, 3) → (3, 3)

Step 3: Now shapes match! Proceed with operation.
  (3, 3) + (3, 3) → (3, 3) ✅
```

---

## Simple Examples — Building Intuition

### Case 1: Scalar + Array (0D + nD)

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

# Scalar: shape ()
# Array:  shape (2, 3)
result = arr + 100

print(result)
# [[101 102 103]
#  [104 105 106]]

# Broadcasting trace:
# scalar: shape ()  → padded to (1, 1) → stretched to (2, 3)
# arr:    shape (2, 3)
# Result: shape (2, 3)
```

### Case 2: 1D Array + 2D Array (Row Broadcasting)

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])  # shape: (3, 3)

row = np.array([10, 20, 30])   # shape: (3,)

result = matrix + row

print(result)
# [[11 22 33]
#  [14 25 36]
#  [17 28 39]]

# Broadcasting trace:
# row:    shape (3,) → padded to (1, 3) → stretched to (3, 3)
# matrix: shape (3, 3)
# Result: shape (3, 3)
#
# Effectively:
# [[10, 20, 30],   ← row broadcasts to each row of matrix
#  [10, 20, 30],
#  [10, 20, 30]]
# + 
# [[1, 2, 3],
#  [4, 5, 6],
#  [7, 8, 9]]
```

### Case 3: Column Broadcasting (Requires Reshape)

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])  # shape: (3, 3)

col = np.array([10, 20, 30])   # shape: (3,)

# ❌ This won't give you column broadcast!
# result = matrix + col  ← This broadcasts row-wise!

# ✅ Reshape col to (3, 1) — a column vector
col_reshaped = col.reshape(3, 1)  # shape: (3, 1)
# Or: col[:, np.newaxis]          # same thing

result = matrix + col_reshaped

print(result)
# [[11 12 13]   ← row 0 + 10
#  [24 25 26]   ← row 1 + 20
#  [37 38 39]]  ← row 2 + 30

# Broadcasting trace:
# col:    shape (3, 1) → stretched to (3, 3)
# matrix: shape (3, 3)
# Result: shape (3, 3)
#
# col_reshaped broadcasts to:
# [[10, 10, 10],
#  [20, 20, 20],
#  [30, 30, 30]]
```

> [!tip] Row vs Column Broadcasting
> - 1D array of shape `(n,)` → broadcasts to match **rows** of an `(m, n)` matrix
> - Column vector of shape `(n, 1)` → broadcasts to match **columns** of an `(n, m)` matrix
> 
> **The key insight:** Use `arr.reshape(-1, 1)` or `arr[:, np.newaxis]` to turn a 1D array into a column vector!

---

## The Rules in Action

### Shape Compatibility Table

Two dimensions are compatible when they are:
- **Equal**, OR
- **One of them is 1**

```
Shape A    Shape B    Compatible?  Result Shape
───────────────────────────────────────────────
(5,)       (5,)       ✅ Equal     (5,)
(1,)       (5,)       ✅ One is 1  (5,)
(3, 5)     (5,)       ✅ Padded    (3, 5)
(3, 5)     (3, 1)     ✅ Col vec   (3, 5)
(3, 5)     (1, 5)     ✅ Row vec   (3, 5)
(3, 5)     (1, 1)     ✅ Scalar    (3, 5)
(3, 5)     (3, 3)     ❌ 5 ≠ 3    ERROR!
(3, 5)     (4, 5)     ❌ 3 ≠ 4    ERROR!
(3, 4, 5)  (4, 5)     ✅ Padded    (3, 4, 5)
(3, 4, 5)  (3, 1, 5)  ✅ Middle 1  (3, 4, 5)
```

### Step-by-Step Rule Application

```python
# Example: shape (4, 3) + shape (3,)
A = np.ones((4, 3))       # shape: (4, 3)
b = np.array([10, 20, 30]) # shape: (3,)

# Step 1 (Rule 1): Pad b with 1s on the LEFT
#   (3,) → (1, 3)

# Step 2 (Rule 2): Stretch the 1 dimension
#   (1, 3) → (4, 3)  ← stretched from 1 to 4 in dim 0

# Step 3: Shapes now match → compute
result = A + b
print(result.shape)  # (4, 3) ✅
print(result)
# [[11. 21. 31.]
#  [11. 21. 31.]
#  [11. 21. 31.]
#  [11. 21. 31.]]
```

```python
# Example: shape (3, 1) + shape (1, 4)
col = np.array([[1], [2], [3]])    # shape: (3, 1)
row = np.array([[10, 20, 30, 40]]) # shape: (1, 4)

# Rule 2: (3, 1) + (1, 4)
#   dim 0: max(3, 1) = 3 → both become (3, _)
#   dim 1: max(1, 4) = 4 → both become (_, 4)
# Result shape: (3, 4)

result = col + row
print(result)
# [[11 21 31 41]   ← 1 + [10, 20, 30, 40]
#  [12 22 32 42]   ← 2 + [10, 20, 30, 40]
#  [13 23 33 43]]  ← 3 + [10, 20, 30, 40]
```

---

## When Broadcasting Fails

```python
# ❌ Shape mismatch that can't be resolved
a = np.ones((3, 4))   # shape: (3, 4)
b = np.ones((4, 3))   # shape: (4, 3)

try:
    result = a + b
except ValueError as e:
    print(f"Error: {e}")
    # Error: operands could not be broadcast together with shapes (3,4) (4,3)

# Why? Dimensions checked right-to-left:
#   dim 1: a=4, b=3 → neither is 1, not equal → FAIL ❌

# ❌ Another common mistake
data   = np.ones((5, 3))   # shape: (5, 3)
labels = np.array([1,2,3,4,5])  # shape: (5,)

try:
    result = data + labels
except ValueError as e:
    print(f"Error: {e}")
    # Error: shapes (5,3) and (5,) don't match
    # labels gets padded to (1,5), but (5,3) vs (1,5): 3 ≠ 5 → FAIL

# ✅ Fix: reshape labels
result = data + labels.reshape(5, 1)  # shape: (5,1) → broadcasts to (5,3)
print(result.shape)  # (5, 3) ✅
```

> [!danger] Broadcasting Error Checklist
> When you get a broadcasting error:
> 1. Print the shapes of both arrays (`arr.shape`)
> 2. Align the shapes from the **right**
> 3. Check if each dimension pair is: equal, or one of them is 1
> 4. Reshape with `.reshape()` or `[:, np.newaxis]` to fix

---

## Real-World Use Cases

### Use Case 1: Center a Dataset (Subtract Mean)

```python
# Dataset: 5 samples, 3 features each
data = np.array([[2.0, 3.0, 1.0],
                 [4.0, 1.0, 5.0],
                 [3.0, 2.0, 4.0],
                 [1.0, 5.0, 2.0],
                 [5.0, 4.0, 3.0]])

# Column means: shape (3,)
col_means = data.mean(axis=0)
print(col_means)  # [3. 3. 3.]

# Subtract mean from each column (center the data)
# data: (5, 3), col_means: (3,) → padded to (1,3) → (5,3)
centered = data - col_means

print(centered)
# [[-1.  0. -2.]
#  [ 1. -2.  2.]
#  [ 0. -1.  1.]
#  [-2.  2. -1.]
#  [ 2.  1.  0.]]

# Verify: all column means should now be ~0
print(centered.mean(axis=0))  # [0. 0. 0.]  ← centered!
```

### Use Case 2: Normalize (Z-score Standardization)

```python
data = np.array([[2.0, 3.0, 1.0],
                 [4.0, 1.0, 5.0],
                 [3.0, 2.0, 4.0]])

mean = data.mean(axis=0)   # shape: (3,)
std  = data.std(axis=0)    # shape: (3,)

# Both mean and std broadcast over rows
normalized = (data - mean) / std

print(normalized.round(3))
print(f"After normalization — Mean: {normalized.mean(axis=0).round(5)}")
print(f"After normalization — Std:  {normalized.std(axis=0).round(5)}")
```

### Use Case 3: Distance Matrix

```python
# Compute pairwise Euclidean distances between points
points = np.array([[0, 0],
                   [1, 0],
                   [0, 1],
                   [1, 1]])  # shape: (4, 2)

# Trick: use broadcasting to get all pairwise differences
# points[:, np.newaxis]:  shape (4, 1, 2)
# points[np.newaxis, :]:  shape (1, 4, 2)
# Difference:             shape (4, 4, 2) via broadcasting

diff = points[:, np.newaxis, :] - points[np.newaxis, :, :]
# diff[i, j] = points[i] - points[j]

distances = np.sqrt((diff**2).sum(axis=-1))
print(distances.round(3))
# [[0.    1.    1.    1.414]
#  [1.    0.    1.414 1.   ]
#  [1.    1.414 0.    1.   ]
#  [1.414 1.    1.    0.   ]]
```

### Use Case 4: Image Processing

```python
# A grayscale image is a 2D array
# A color image is a 3D array: (height, width, 3) for RGB channels

# Simulate an RGB image (height=4, width=4, channels=3)
image = np.random.randint(0, 256, (4, 4, 3), dtype=np.uint8)
print(image.shape)  # (4, 4, 3)

# Apply per-channel scaling (e.g., increase blue channel)
channel_scale = np.array([0.8, 0.9, 1.2])  # R, G, B scaling
# channel_scale shape: (3,) → broadcasts to (4, 4, 3)

scaled = image * channel_scale
print(scaled.shape)  # (4, 4, 3) ✅

# Apply grayscale using luminance formula
# Standard weights for R, G, B → Luminance
weights = np.array([0.2989, 0.5870, 0.1140])
grayscale = (image * weights).sum(axis=2)  # Sum along channel axis
print(grayscale.shape)  # (4, 4)  ← grayscale image!
```

### Use Case 5: Outer Product Without np.outer

```python
# Broadcasting to create all combinations (outer product)
a = np.array([1, 2, 3])          # shape: (3,)
b = np.array([10, 20, 30, 40])    # shape: (4,)

# Reshape a to column vector: (3, 1)
# b stays as row vector: (4,)
outer = a[:, np.newaxis] * b      # (3, 1) * (4,) → (3, 4)

print(outer)
# [[ 10  20  30  40]   ← 1 × [10,20,30,40]
#  [ 20  40  60  80]   ← 2 × [10,20,30,40]
#  [ 30  60  90 120]]  ← 3 × [10,20,30,40]
```

---

## Advanced Broadcasting

### `np.newaxis` (same as `None`)

`np.newaxis` inserts a new dimension of size 1, making it easy to control broadcasting direction:

```python
arr = np.array([1, 2, 3, 4])  # shape: (4,)

# Add dimension as row: shape (1, 4)
row_vec = arr[np.newaxis, :]   # or arr[None, :]
print(row_vec.shape)  # (1, 4)

# Add dimension as column: shape (4, 1)
col_vec = arr[:, np.newaxis]   # or arr[:, None]
print(col_vec.shape)  # (4, 1)

# Outer product with newaxis
print(col_vec * row_vec)  # (4, 1) * (1, 4) → (4, 4)
# [[ 1  2  3  4]
#  [ 2  4  6  8]
#  [ 3  6  9 12]
#  [ 4  8 12 16]]
```

### Broadcasting with 3D Arrays

```python
# Shape: (batch_size, rows, cols)
batch = np.ones((10, 4, 4))  # 10 matrices, each 4×4

# Bias to add: different bias per row
row_bias = np.array([1, 2, 3, 4])   # shape: (4,)

# We want to add to each matrix in the batch
# batch:    (10, 4, 4)
# row_bias: (    4,  )  → padded to (1, 1, 4) → (10, 4, 4)
result = batch + row_bias

# To add a different value per matrix (batch-wise):
batch_bias = np.arange(10).reshape(10, 1, 1)  # shape: (10, 1, 1)
result2 = batch + batch_bias  # → (10, 4, 4)
```

---

## Broadcasting Mental Model

> [!tip] The "Stretching" Mental Model
>
> Think of broadcasting as **conceptually stretching** smaller arrays to fill the shape of the larger one — but without actually allocating memory for the repeated data.
>
> ```
> Row vector:    [1, 2, 3]    shape (1, 3)
> "Stretched":   [1, 2, 3]
>                [1, 2, 3]    shape (3, 3) ← conceptual only!
>                [1, 2, 3]
>
> Col vector:    [1]          shape (3, 1)
>                [2]
>                [3]
> "Stretched":   [1, 1, 1]    shape (3, 3) ← conceptual only!
>                [2, 2, 2]
>                [3, 3, 3]
> ```

---

## Summary

> [!success] Key Takeaways
>
> **The 3 Rules (memorize these!):**
> 1. If ranks differ → **pad smaller shape with 1s on the LEFT**
> 2. Dimensions of size 1 → **stretched to match the other array**
> 3. Non-1 size mismatch → **ValueError** (no broadcast possible)
>
> **Common patterns:**
> ```python
> # Row-wise broadcast
> matrix + row_vec              # (m, n) + (n,) → (m, n)
> 
> # Column-wise broadcast
> matrix + col_vec              # (m, n) + (m, 1) → (m, n)
> 
> # Create column vector from 1D
> arr[:, np.newaxis]            # (n,) → (n, 1)
> 
> # Outer product
> col[:, np.newaxis] * row      # (m, 1) * (n,) → (m, n)
> ```
>
> **Real-world applications:**
> - Centering/normalizing datasets
> - Distance matrices
> - Image processing
> - Feature scaling in ML

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[04-vectorization]] | [[06-numpy-exercises]] |

---

*Tags: #numpy #broadcasting #shapes #newaxis #vectorization #normalization*