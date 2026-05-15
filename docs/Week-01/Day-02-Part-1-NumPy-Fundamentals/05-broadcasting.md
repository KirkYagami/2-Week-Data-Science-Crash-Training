# Broadcasting
## Operating on Arrays of Different Shapes

Broadcasting is the mechanism that lets NumPy perform arithmetic between arrays of different shapes. Without it, every operation would require arrays of exactly the same size, and you would need to manually replicate data to make shapes match — wasting memory and writing boilerplate. With it, you can normalize a dataset of 10,000 rows with a single expression. The rules are strict and learnable. Once they click, they never break.

## Learning Objectives

- Explain the three broadcasting rules from memory and apply them step-by-step
- Predict the output shape of any broadcasting operation by aligning shapes from the right
- Use `np.newaxis` and `reshape` to control which direction broadcasting happens
- Recognize when broadcasting will fail and fix the shape mismatch
- Apply broadcasting to realistic patterns: row normalization, column subtraction, outer products, distance matrices

---

## The Core Idea

Broadcasting is NumPy's way of "stretching" a smaller array to match a larger one during arithmetic — conceptually, not in memory. No data is duplicated; NumPy computes as if it were.

```python
import numpy as np

# The simplest case: scalar + array
arr = np.array([1, 2, 3, 4, 5])
result = arr + 10

# Conceptually NumPy treats the scalar as [10, 10, 10, 10, 10]
# But it does NOT create that array — it computes in-place
print(result)  # Output: [11 12 13 14 15]
```

The rule is: if NumPy can figure out how to stretch one array to match the other without ambiguity, it does so. If it cannot, it raises a `ValueError`.

---

## The Three Broadcasting Rules

These rules are applied every time you combine two arrays with different shapes. Learn them precisely — they explain every broadcasting result and every broadcasting error.

> [!info] The Three Rules
>
> **Rule 1:** If the arrays have different numbers of dimensions, pad the smaller shape with 1s **on the left** until both shapes have the same length.
>
> **Rule 2:** Any dimension of size 1 is stretched to match the corresponding dimension of the other array.
>
> **Rule 3:** If two corresponding dimensions are different and **neither is 1**, broadcasting fails with a `ValueError`.

### Applying the Rules: Step-by-Step

```
Example: (3,) + (3, 3)

Step 0 — starting shapes:
  A: (3,)
  B: (3, 3)

Step 1 — Rule 1: pad smaller shape with 1s on the LEFT:
  A: (1, 3)
  B: (3, 3)

Step 2 — Rule 2: stretch size-1 dimensions:
  A: (3, 3)   ← the 1 stretched to 3
  B: (3, 3)

Step 3 — Rule 3: no mismatches → compute
  Result shape: (3, 3)  ✓
```

```
Example: (3, 1) + (1, 4)

Step 0:  A: (3, 1)   B: (1, 4)
Step 1:  Both already have 2 dims — no padding needed
Step 2:  dim 0: max(3, 1) = 3  → A keeps 3, B stretches from 1 to 3
         dim 1: max(1, 4) = 4  → A stretches from 1 to 4, B keeps 4
Result shape: (3, 4)  ✓
```

```
Example: (3, 4) + (4, 3)

Step 0:  A: (3, 4)   B: (4, 3)
Step 1:  Both have 2 dims — no padding
Step 2:  dim 0: 3 vs 4 — neither is 1 → FAIL
         dim 1: 4 vs 3 — neither is 1 → FAIL
Result: ValueError  ✗
```

---

## Visual Intuition: Think "Stretching"

The most useful mental model: broadcasting conceptually *tiles* the smaller array to fill the larger one's shape, without actually allocating the memory.

```
Row broadcasting: (3, 1) and (3, 4)

(3, 1):   [10]          tiles →   [10  10  10  10]
          [20]                    [20  20  20  20]
          [30]                    [30  30  30  30]

(3, 4):   [ 1   2   3   4]
          [ 5   6   7   8]
          [ 9  10  11  12]

Result:   [11  12  13  14]
          [25  26  27  28]
          [39  40  41  42]
```

```
Column broadcasting: (1, 4) and (3, 4)

(1, 4):   [10  20  30  40]   tiles →   [10  20  30  40]
                                        [10  20  30  40]
                                        [10  20  30  40]
```

---

## Case 1: Row Broadcasting (the Common Case)

A 1-D array broadcasts against rows of a 2-D array — it is applied to each row.

```python
import numpy as np

matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])   # shape: (3, 3)

row = np.array([10, 20, 30])     # shape: (3,)

result = matrix + row
print(result)
# Shape trace: (3,) → padded to (1, 3) → stretched to (3, 3)
# row becomes [[10, 20, 30],
#              [10, 20, 30],
#              [10, 20, 30]]  ← conceptually
# Output:
# [[11 22 33]
#  [14 25 36]
#  [17 28 39]]
```

---

## Case 2: Column Broadcasting (Requires Reshape)

To broadcast a 1-D array against columns, you must turn it into a column vector with shape `(n, 1)`.

```python
import numpy as np

matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])   # shape: (3, 3)

col = np.array([10, 20, 30])     # shape: (3,)

# This does NOT do column broadcasting — col gets treated as a row vector!
# (3,) → padded to (1, 3) → stretched to (3, 3) → adds to each ROW
wrong = matrix + col
print(wrong)
# Output:
# [[11 22 33]    ← each element got [10, 20, 30] added row-wise
#  [14 25 36]
#  [17 28 39]]

# To broadcast per-column, reshape to (3, 1)
col_vec = col.reshape(3, 1)   # or: col[:, np.newaxis]
correct = matrix + col_vec
print(correct)
# Shape trace: (3, 1) → stretched to (3, 3)
# col_vec becomes [[10, 10, 10],
#                  [20, 20, 20],
#                  [30, 30, 30]]  ← conceptually
# Output:
# [[11 12 13]    ← row 0 + 10
#  [24 25 26]    ← row 1 + 20
#  [37 38 39]]   ← row 2 + 30
```

> [!warning] The Most Common Broadcasting Confusion
> A 1-D array of shape `(n,)` always pads to `(1, n)` — a row vector. It broadcasts against every row of a matrix, not every column. If you want to add different values to each *row* (treating each row as a unit), you need a column vector of shape `(n, 1)`. Always ask yourself: "Am I adding per-row or per-column?" Then check your shapes.

---

## `np.newaxis` — Controlling the Direction

`np.newaxis` (identical to `None`) inserts a size-1 dimension at the specified position. It is the cleanest way to control broadcasting direction without hardcoding shape numbers.

```python
import numpy as np

arr = np.array([1, 2, 3, 4])  # shape: (4,)

# Add axis at position 0 → becomes a row vector
row_vec = arr[np.newaxis, :]
print(row_vec.shape)  # Output: (1, 4)

# Add axis at position 1 → becomes a column vector
col_vec = arr[:, np.newaxis]
print(col_vec.shape)  # Output: (4, 1)

# Outer product via broadcasting: (4, 1) * (1, 4) → (4, 4)
outer = col_vec * row_vec
print(outer)
# Output:
# [[ 1  2  3  4]    ← 1 × [1, 2, 3, 4]
#  [ 2  4  6  8]    ← 2 × [1, 2, 3, 4]
#  [ 3  6  9 12]    ← 3 × [1, 2, 3, 4]
#  [ 4  8 12 16]]   ← 4 × [1, 2, 3, 4]
```

---

## Shape Compatibility Reference

Shapes are compared **from right to left**. Dimensions are compatible if they are equal, or one of them is 1.

```
Shape A       Shape B       Compatible?   Result Shape
─────────────────────────────────────────────────────
(5,)          (5,)          ✓ equal       (5,)
(1,)          (5,)          ✓ one is 1    (5,)
(3, 5)        (5,)          ✓ padded      (3, 5)
(3, 5)        (3, 1)        ✓ col vec     (3, 5)
(3, 5)        (1, 5)        ✓ row vec     (3, 5)
(3, 5)        (1, 1)        ✓ scalar      (3, 5)
(3, 5)        (3, 3)        ✗ 5 ≠ 3       ValueError
(3, 5)        (4, 5)        ✗ 3 ≠ 4       ValueError
(3, 4, 5)     (4, 5)        ✓ padded      (3, 4, 5)
(3, 4, 5)     (3, 1, 5)     ✓ middle 1    (3, 4, 5)
```

---

## When Broadcasting Fails

```python
import numpy as np

# Incompatible shapes — neither non-1 dimension matches
a = np.ones((3, 4))
b = np.ones((4, 3))
try:
    result = a + b
except ValueError as e:
    print(e)
# Output: operands could not be broadcast together with shapes (3,4) (4,3)
# Why: rightmost dims are 4 and 3 — not equal, neither is 1

# Common mistake: confusing row vs column intent
data   = np.ones((5, 3))
labels = np.array([1, 2, 3, 4, 5])   # shape: (5,)
try:
    result = data + labels
except ValueError as e:
    print(e)
# Output: operands could not be broadcast together with shapes (5,3) (5,)
# Why: (5,) padded to (1, 5), then compared with (5, 3):
#       dim 0: 5 vs 1 → stretched to 5 ✓
#       dim 1: 3 vs 5 → neither is 1 ✗

# Fix: make labels a column vector
result = data + labels.reshape(5, 1)   # shape (5, 1) → broadcasts to (5, 3)
print(result.shape)  # Output: (5, 3)  ✓
```

> [!warning] How to Debug a Broadcasting Error
> When you see "operands could not be broadcast together with shapes X Y":
> 1. Print both shapes: `print(a.shape, b.shape)`
> 2. Write them out aligned from the right
> 3. Check each dimension pair: equal, or one is 1?
> 4. Find the first mismatch and decide: do you want row-wise or column-wise behavior?
> 5. Reshape with `.reshape(-1, 1)` or `[:, np.newaxis]` to fix

---

## Practical Patterns

### Subtract Column Means (Center the Data)

```python
import numpy as np

# 5 samples, 3 features
data = np.array([[2.0, 30.0, 1.0],
                 [4.0, 10.0, 5.0],
                 [3.0, 20.0, 4.0],
                 [1.0, 50.0, 2.0],
                 [5.0, 40.0, 3.0]])

# Column means — shape (3,)
col_means = data.mean(axis=0)
print(col_means)  # Output: [3. 30.  3.]

# Subtract: (5, 3) - (3,) → (3,) padded to (1, 3) → (5, 3) ✓
centered = data - col_means
print(centered)
# Output:
# [[-1.   0.  -2.]
#  [ 1. -20.   2.]
#  [ 0. -10.   1.]
#  [-2.  20.  -1.]
#  [ 2.  10.   0.]]

print(centered.mean(axis=0))  # Output: [0. 0. 0.]  ← centered!
```

### Z-score Standardization

```python
import numpy as np

data = np.array([[2.0, 30.0, 1.0],
                 [4.0, 10.0, 5.0],
                 [3.0, 20.0, 4.0]])

mean = data.mean(axis=0)   # shape: (3,)
std  = data.std(axis=0)    # shape: (3,)

# Both mean and std broadcast over rows of data
standardized = (data - mean) / std
print(standardized.round(3))
print(standardized.mean(axis=0).round(8))  # Output: [0. 0. 0.]
print(standardized.std(axis=0).round(8))   # Output: [1. 1. 1.]
```

### Normalize Rows to Sum to 1 (Softmax Preparation)

```python
import numpy as np

# Each row is a vector of unnormalized scores
scores = np.array([[1.0, 2.0, 3.0],
                   [4.0, 0.0, 2.0],
                   [1.0, 1.0, 1.0]])

# Row sums: shape (3,)
row_sums = scores.sum(axis=1)
print(row_sums)  # Output: [6. 6. 3.]

# To divide each row by its sum, row_sums must be (3, 1) not (3,)
normalized = scores / row_sums[:, np.newaxis]
print(normalized.round(3))
# Output:
# [[0.167 0.333 0.5  ]
#  [0.667 0.    0.333]
#  [0.333 0.333 0.333]]

print(normalized.sum(axis=1))  # Output: [1. 1. 1.]  ← rows sum to 1
```

### Add Per-Row Bias to a Matrix

```python
import numpy as np

# A matrix where each row represents a sample
weights = np.array([[0.5, 1.2, -0.3],
                    [0.8, -0.4, 1.1],
                    [-0.2, 0.9, 0.7]])

# A different bias value per row
bias = np.array([0.1, 0.2, 0.3])   # shape: (3,)

# bias broadcasts as a row vector → adds per-column, not per-row
# To add per-row: reshape to column vector
result = weights + bias[:, np.newaxis]   # (3, 1) → (3, 3)
print(result.round(2))
# Output:
# [[0.6  1.3  -0.2]   ← row 0 + 0.1
#  [1.0  -0.2  1.3]   ← row 1 + 0.2
#  [0.1   1.2  1.0]]  ← row 2 + 0.3
```

### Pairwise Distance Matrix

This is a classic broadcasting trick — computing every pair of distances in one expression.

```python
import numpy as np

# 4 points in 2-D space
points = np.array([[0, 0],
                   [1, 0],
                   [0, 1],
                   [1, 1]])   # shape: (4, 2)

# Expand dims to (4, 1, 2) and (1, 4, 2)
# Their difference broadcasts to (4, 4, 2)
diff = points[:, np.newaxis, :] - points[np.newaxis, :, :]
# diff[i, j] = points[i] - points[j]

distances = np.sqrt((diff ** 2).sum(axis=-1))
print(distances.round(3))
# Output:
# [[0.    1.    1.    1.414]
#  [1.    0.    1.414 1.   ]
#  [1.    1.414 0.    1.   ]
#  [1.414 1.    1.    0.   ]]
# distances[i, j] = Euclidean distance from point i to point j
```

### Image Processing: Channel Scaling

```python
import numpy as np
rng = np.random.default_rng(42)

# An RGB image: shape (height, width, 3)
image = rng.integers(0, 256, (64, 64, 3), dtype=np.uint8)

# Boost the blue channel, reduce the red channel
channel_scale = np.array([0.7, 0.9, 1.3])   # shape: (3,)
# (64, 64, 3) * (3,) → (3,) padded to (1, 1, 3) → (64, 64, 3) ✓

scaled = (image.astype(np.float32) * channel_scale)
scaled = scaled.clip(0, 255).astype(np.uint8)
print(scaled.shape)   # Output: (64, 64, 3)

# Convert to grayscale using luminance weights
luminance_weights = np.array([0.2989, 0.5870, 0.1140])
gray = (image * luminance_weights).sum(axis=2)
print(gray.shape)  # Output: (64, 64)  ← grayscale
```

---

## 3-D Broadcasting in Batch Operations

```python
import numpy as np

# Batch of 10 matrices, each 4×5
batch = np.ones((10, 4, 5))   # shape: (10, 4, 5)

# Add a different scalar offset to each matrix in the batch
offsets = np.arange(10).reshape(10, 1, 1)   # shape: (10, 1, 1)
result = batch + offsets   # (10, 4, 5) + (10, 1, 1) → (10, 4, 5) ✓

# Add a per-row bias (same bias for all matrices in batch)
row_bias = np.array([1, 2, 3, 4]).reshape(1, 4, 1)   # shape: (1, 4, 1)
result = batch + row_bias   # (10, 4, 5) + (1, 4, 1) → (10, 4, 5) ✓
```

---

> [!success] Key Takeaways
> - Rule 1: pad smaller shape with 1s on the **left** to match dimension count
> - Rule 2: size-1 dimensions **stretch** to match the other array's size
> - Rule 3: mismatched non-1 dimensions → `ValueError`
> - A 1-D array `(n,)` acts as a **row vector** `(1, n)` — it broadcasts over rows of a matrix
> - To broadcast over columns, use `arr[:, np.newaxis]` or `arr.reshape(-1, 1)` for shape `(n, 1)`
> - When debugging a shape error: print both shapes, align from the right, find the mismatch
> - Key patterns: center data with `data - data.mean(axis=0)`, normalize rows with `/ row_sums[:, np.newaxis]`, compute outer products with `col[:, np.newaxis] * row`

---

[[04-vectorization]] | [[06-numpy-exercises]]
