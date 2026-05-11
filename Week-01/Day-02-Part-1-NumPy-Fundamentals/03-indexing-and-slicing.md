# 03 — Indexing & Slicing
## Accessing and Modifying Array Data

> [!quote] "If array creation is building the house, indexing is knowing which room to walk into."

---

## 🧭 Table of Contents

- [[#Core Concept: Zero-Based Indexing]]
- [[#1D Array Indexing]]
- [[#1D Array Slicing]]
- [[#2D Array Indexing]]
- [[#2D Array Slicing]]
- [[#3D Array Indexing]]
- [[#Boolean Indexing (Masking)]]
- [[#Fancy Indexing]]
- [[#Modifying Arrays via Indexing]]
- [[#Views vs Copies — Critical Concept]]
- [[#Summary]]

---

## Core Concept: Zero-Based Indexing

NumPy, like Python, uses **zero-based indexing** — the first element is at position 0.

```
Array:    [10, 20, 30, 40, 50]
Index:     0   1   2   3   4    ← Positive indices (left to right)
Negative: -5  -4  -3  -2  -1   ← Negative indices (right to left)
```

```python
import numpy as np

arr = np.array([10, 20, 30, 40, 50])

# Positive indexing
print(arr[0])   # 10  ← First element
print(arr[2])   # 30  ← Third element
print(arr[4])   # 50  ← Last element

# Negative indexing (count from end)
print(arr[-1])  # 50  ← Last element
print(arr[-2])  # 40  ← Second to last
print(arr[-5])  # 10  ← Same as arr[0]
```

---

## 1D Array Indexing

### Single Element Access

```python
arr = np.array([100, 200, 300, 400, 500])

# Get elements
first   = arr[0]    # 100
last    = arr[-1]   # 500
middle  = arr[2]    # 300

# Modify a single element
arr[0] = 999
print(arr)  # [999 200 300 400 500]
```

---

## 1D Array Slicing

Slicing uses the syntax: `arr[start : stop : step]`

**Rules:**
- `start` is **included** (default: 0)
- `stop` is **excluded** (default: length of array)
- `step` is the increment (default: 1)

```python
arr = np.array([0, 10, 20, 30, 40, 50, 60, 70, 80, 90])
#              [0   1   2   3   4   5   6   7   8   9]  ← indices

# Basic slices
print(arr[2:5])     # [20 30 40]  ← indices 2, 3, 4 (NOT 5)
print(arr[:4])      # [ 0 10 20 30]  ← from start to index 3
print(arr[6:])      # [60 70 80 90]  ← from index 6 to end
print(arr[:])       # all elements (copy of whole array)

# With step
print(arr[::2])     # [ 0 20 40 60 80]  ← every 2nd element
print(arr[1::2])    # [10 30 50 70 90]  ← every 2nd, starting at index 1
print(arr[2:8:3])   # [20 50]           ← indices 2, 5 (step of 3)

# Negative step (reverse)
print(arr[::-1])    # [90 80 70 60 50 40 30 20 10  0]  ← reversed!
print(arr[8:2:-2])  # [80 60 40]  ← from index 8, going backward, step 2

# Negative indices in slices
print(arr[-3:])     # [70 80 90]  ← last 3 elements
print(arr[:-3])     # [ 0 10 20 30 40 50 60]  ← everything except last 3
print(arr[-5:-2])   # [50 60 70]  ← middle section
```

### 🧠 Slice Visualization

```
arr = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]

arr[2:7]:
     ┌────────────────┐
[0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
         ↑              ↑
       start=2        stop=7 (excluded)
Result: [20, 30, 40, 50, 60]

arr[::2]:
[0,  10, 20,  30, 40,  50, 60,  70, 80,  90]
 ↑        ↑        ↑        ↑        ↑
Result: [0, 20, 40, 60, 80]
```

---

## 2D Array Indexing

For 2D arrays (matrices), indexing uses **two indices**: `arr[row, col]`

```python
matrix = np.array([[11, 12, 13, 14],
                   [21, 22, 23, 24],
                   [31, 32, 33, 34]])
#                   col0 col1 col2 col3
# row 0:           [11,  12,  13,  14]
# row 1:           [21,  22,  23,  24]
# row 2:           [31,  32,  33,  34]

# Single element: [row, col]
print(matrix[0, 0])   # 11  ← top-left
print(matrix[1, 2])   # 23  ← row 1, column 2
print(matrix[2, 3])   # 34  ← bottom-right
print(matrix[-1, -1]) # 34  ← same as above using negatives

# Entire row
print(matrix[0])      # [11 12 13 14]  ← first row
print(matrix[1, :])   # [21 22 23 24]  ← same with explicit slice

# Entire column
print(matrix[:, 0])   # [11 21 31]  ← first column
print(matrix[:, 2])   # [13 23 33]  ← third column
```

### Visual Reference

```
matrix = [[11, 12, 13, 14],
          [21, 22, 23, 24],
          [31, 32, 33, 34]]

matrix[1, 2] → row 1, col 2 → 23

matrix[0]    → entire row 0 → [11, 12, 13, 14]
matrix[:, 1] → entire col 1 → [12, 22, 32]
```

---

## 2D Array Slicing

Slicing works on both dimensions simultaneously:

```python
matrix = np.array([[11, 12, 13, 14],
                   [21, 22, 23, 24],
                   [31, 32, 33, 34],
                   [41, 42, 43, 44]])

# Submatrix: rows 0-1, columns 1-2
sub = matrix[0:2, 1:3]
print(sub)
# [[12 13]
#  [22 23]]

# Top-left 2x2
print(matrix[:2, :2])
# [[11 12]
#  [21 22]]

# Bottom-right 2x2
print(matrix[2:, 2:])
# [[33 34]
#  [43 44]]

# All rows, skip every other column
print(matrix[:, ::2])
# [[11 13]
#  [21 23]
#  [31 33]
#  [41 43]]

# Reverse row order
print(matrix[::-1, :])
# [[41 42 43 44]
#  [31 32 33 34]
#  [21 22 23 24]
#  [11 12 13 14]]

# Reverse column order
print(matrix[:, ::-1])
# [[14 13 12 11]
#  [24 23 22 21]
#  [34 33 32 31]
#  [44 43 42 41]]

# Rotate 180° (reverse both)
print(matrix[::-1, ::-1])
# [[44 43 42 41]
#  [34 33 32 31]
#  [24 23 22 21]
#  [14 13 12 11]]
```

### 🧠 2D Slice Mental Model

```
matrix[row_slice, col_slice]

Think of it as: "give me rows [r1:r2] and columns [c1:c2]"

matrix[1:3, 1:3]:
        col1 col2
row 1 → [22, 23]
row 2 → [32, 33]
```

---

## 3D Array Indexing

```python
# Think of 3D as a "stack of matrices"
# Shape (depth, rows, cols) = (layers, rows, cols)

cube = np.array([[[1, 2, 3],    # Layer 0
                  [4, 5, 6],
                  [7, 8, 9]],
                 [[10, 11, 12], # Layer 1
                  [13, 14, 15],
                  [16, 17, 18]]])

print(cube.shape)   # (2, 3, 3) → 2 layers, 3 rows, 3 cols

# Access: [layer, row, col]
print(cube[0, 0, 0])    # 1   ← layer 0, row 0, col 0
print(cube[1, 2, 2])    # 18  ← layer 1, row 2, col 2
print(cube[0, :, 1])    # [2 5 8]  ← layer 0, all rows, column 1
print(cube[:, 1, :])    # Layer slice — row 1 from each layer
# [[ 4  5  6]
#  [13 14 15]]
```

---

## Boolean Indexing (Masking)

Boolean indexing lets you **filter** elements based on a condition. This is extremely powerful and common in data science!

```python
arr = np.array([15, 3, 42, 8, 27, 11, 56, 4, 33])

# Step 1: Create a boolean mask
mask = arr > 20
print(mask)  # [False False  True False  True False  True False  True]

# Step 2: Use mask to filter
result = arr[mask]
print(result)  # [42 27 56 33]

# One-liner (most common in practice)
print(arr[arr > 20])   # [42 27 56 33]
print(arr[arr < 10])   # [3 8 4]
print(arr[arr == 42])  # [42]

# Combining conditions
# Use & (and), | (or), ~ (not) — NOT Python's 'and'/'or'/'not'!
print(arr[(arr > 10) & (arr < 30)])  # [15 27 11]   AND condition
print(arr[(arr < 5)  | (arr > 50)])  # [ 3  4 56]   OR condition
print(arr[~(arr > 20)])              # [15  3  8 11  4]  NOT condition
```

### Boolean Indexing on 2D Arrays

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

# Get all elements greater than 5 (returns 1D array!)
print(matrix[matrix > 5])   # [6 7 8 9]

# Replace values meeting condition
matrix[matrix < 5] = 0
print(matrix)
# [[0 0 3]
#  [4 5 6]
#  [7 8 9]]

# Select entire rows based on a condition
data = np.array([[1, 80],   # [id, score]
                 [2, 45],
                 [3, 92],
                 [4, 60],
                 [5, 78]])

# Get rows where score > 70
passed = data[data[:, 1] > 70]
print(passed)
# [[ 1 80]
#  [ 3 92]
#  [ 5 78]]
```

### `np.where()` — Conditional Selection

```python
arr = np.array([10, -5, 3, -8, 7, -2])

# Replace negative values with 0
result = np.where(arr > 0, arr, 0)
print(result)  # [10  0  3  0  7  0]

# np.where(condition, value_if_true, value_if_false)
grades = np.array([85, 42, 91, 55, 73, 38])
labels = np.where(grades >= 60, "Pass", "Fail")
print(labels)  # ['Pass' 'Fail' 'Pass' 'Fail' 'Pass' 'Fail']
```

---

## Fancy Indexing

Fancy indexing = using an **array of indices** to select elements.

```python
arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

# Index with a list of positions
indices = [0, 2, 5, 8]
print(arr[indices])   # [10 30 60 90]

# Get elements in a specific order
print(arr[[4, 1, 7]])  # [50 20 80]

# Repeat indices
print(arr[[0, 0, 2, 2]])  # [10 10 30 30]
```

### Fancy Indexing on 2D Arrays

```python
matrix = np.array([[11, 12, 13],
                   [21, 22, 23],
                   [31, 32, 33],
                   [41, 42, 43]])

# Select specific rows
print(matrix[[0, 2], :])    # Rows 0 and 2
# [[11 12 13]
#  [31 32 33]]

# Select specific rows and columns (paired)
rows = [0, 1, 2]
cols = [0, 1, 2]
print(matrix[rows, cols])  # [11 22 33]  ← diagonal! (NOT a submatrix)

# To get a submatrix with fancy indexing, use np.ix_
row_idx = [0, 2]
col_idx = [1, 2]
print(matrix[np.ix_(row_idx, col_idx)])
# [[12 13]
#  [32 33]]
```

---

## Modifying Arrays via Indexing

You can **modify** array values using any indexing technique:

```python
arr = np.zeros(10, dtype=int)
print(arr)  # [0 0 0 0 0 0 0 0 0 0]

# Single element
arr[3] = 99
print(arr)  # [0 0 0 99 0 0 0 0 0 0]

# Slice
arr[5:8] = 7
print(arr)  # [0 0 0 99 0 7 7 7 0 0]

# Boolean mask
arr[arr == 0] = -1
print(arr)  # [-1 -1 -1 99 -1 7 7 7 -1 -1]

# Fancy indexing
arr[[0, 2, 4]] = [100, 200, 300]
print(arr)  # [100 -1 200 99 300 7 7 7 -1 -1]
```

---

## Views vs Copies — Critical Concept

> [!danger] This is One of NumPy's Biggest Gotchas!

When you **slice** a NumPy array, you get a **VIEW** — not a copy. Modifying the slice **modifies the original array!**

```python
original = np.array([1, 2, 3, 4, 5])

# Slicing creates a VIEW
slice_view = original[1:4]
print(slice_view)  # [2 3 4]

# Modify the slice
slice_view[0] = 999
print(slice_view)   # [999 3 4]
print(original)     # [1 999 3 4 5]  ← ORIGINAL IS CHANGED!
```

### When Do You Get a View vs Copy?

| Operation | Result | Modifies Original? |
|-----------|--------|-------------------|
| Basic slicing `arr[1:4]` | **View** | ✅ Yes |
| Fancy indexing `arr[[1,2,3]]` | **Copy** | ❌ No |
| Boolean indexing `arr[arr>0]` | **Copy** | ❌ No |
| `arr.reshape()` | Usually **View** | ✅ Yes |
| `arr.flatten()` | **Copy** | ❌ No |
| `arr.ravel()` | Usually **View** | ✅ Yes |

### How to Force a Copy

```python
original = np.array([1, 2, 3, 4, 5])

# Force a copy using .copy()
safe_copy = original[1:4].copy()
safe_copy[0] = 999
print(safe_copy)  # [999 3 4]
print(original)   # [1 2 3 4 5]  ← Original is SAFE

# Check if an array is a view
print(safe_copy.base is None)     # True → it's a COPY
print(original[1:4].base is None) # False → it's a VIEW
```

> [!tip] Rule of Thumb
> When in doubt, use `.copy()`. The slight memory overhead is worth avoiding subtle bugs that are very hard to track down!

---

## Summary

> [!success] Key Takeaways
>
> 1. **Zero-based indexing** — `arr[0]` is first, `arr[-1]` is last
> 2. **Slicing** = `arr[start:stop:step]` — stop is **excluded**
> 3. **2D indexing** = `matrix[row, col]` or `matrix[row_slice, col_slice]`
> 4. **Boolean indexing** — filter with conditions: `arr[arr > 5]`
> 5. **Combine conditions** with `&`, `|`, `~` (not `and`, `or`, `not`!)
> 6. **`np.where()`** = if-else for arrays
> 7. **Fancy indexing** — use a list of indices: `arr[[0, 3, 7]]`
> 8. **Views vs Copies** — slices are views! Use `.copy()` to be safe
>
> ```python
> # The golden rule:
> arr[2]         # single element
> arr[1:5]       # slice (VIEW!)
> arr[arr > 0]   # boolean filter (copy)
> arr[[1,3,5]]   # fancy indexing (copy)
> arr[1:5].copy() # forced copy
> ```

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[02-array-creation]] | [[04-vectorization]] |

---

*Tags: #numpy #indexing #slicing #boolean-indexing #fancy-indexing #views #masking*