# ⚡ 07 — NumPy Cheat Sheet
## Quick Reference for NumPy Fundamentals

> [!tip] How to Use This
> Keep this file open while solving NumPy exercises. It is designed for fast lookup, not deep explanation.

---

## 📦 Import Convention

```python
import numpy as np
```

This is the standard convention used in almost every Data Science project.

---

## 🧠 Core Ideas

| Concept | Meaning |
|--------|---------|
| `ndarray` | NumPy's main N-dimensional array object |
| Shape | Size of each dimension, e.g. `(3, 4)` |
| Dimension / Axis | Direction in the array: rows, columns, depth |
| `dtype` | Data type of array values, e.g. `int64`, `float64` |
| Vectorization | Applying operations to whole arrays without Python loops |
| Broadcasting | Operating on arrays with different but compatible shapes |

### Check Array Basics

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

print(arr.shape)   # (2, 3)
print(arr.ndim)    # 2
print(arr.size)    # 6
print(arr.dtype)   # int64 or int32
```

---

## 🏗️ Array Creation

### From Python Lists

```python
np.array([1, 2, 3])
np.array([[1, 2, 3], [4, 5, 6]])
```

### Common Constructors

```python
np.zeros(5)              # [0. 0. 0. 0. 0.]
np.ones((2, 3))          # 2x3 array of ones
np.full((2, 3), 7)       # 2x3 array filled with 7
np.eye(3)                # 3x3 identity matrix
np.empty((2, 2))         # uninitialized values
```

### Ranges and Sequences

```python
np.arange(0, 10, 2)      # [0 2 4 6 8]
np.linspace(0, 1, 5)     # [0.   0.25 0.5  0.75 1.  ]
```

### Random Arrays

```python
np.random.seed(42)

np.random.rand(3)             # uniform values in [0, 1)
np.random.randn(3)            # normal distribution
np.random.randint(1, 10, 5)   # random integers
np.random.choice([10, 20, 30], size=4)
```

### Create Like Another Array

```python
arr = np.array([[1, 2], [3, 4]])

np.zeros_like(arr)
np.ones_like(arr)
np.full_like(arr, 99)
```

---

## 🔢 Data Types

```python
arr = np.array([1, 2, 3], dtype=np.float64)
arr.astype(np.int32)
```

| dtype | Use Case |
|------|----------|
| `int32`, `int64` | Whole numbers |
| `float32`, `float64` | Decimal numbers |
| `bool` | Boolean masks |
| `str_` | Text values, less common in NumPy |

> [!warning] Data Science Note
> Most ML libraries expect numeric arrays. Convert strings or mixed data before modeling.

---

## 📐 Reshape and Flatten

```python
arr = np.arange(12)

arr.reshape(3, 4)        # shape (3, 4)
arr.reshape(2, 2, 3)     # shape (2, 2, 3)
arr.reshape(-1, 3)       # infer rows automatically
arr.flatten()            # copy as 1D
arr.ravel()              # view as 1D when possible
```

### Transpose

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

matrix.T
```

---

## 🎯 Indexing and Slicing

### 1D Arrays

```python
arr = np.array([10, 20, 30, 40, 50])

arr[0]       # 10
arr[-1]      # 50
arr[1:4]     # [20 30 40]
arr[:3]      # [10 20 30]
arr[::2]     # [10 30 50]
```

### 2D Arrays

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

matrix[0, 0]       # 1
matrix[1, 2]       # 6
matrix[0, :]       # first row
matrix[:, 1]       # second column
matrix[0:2, 1:3]   # rows 0-1, columns 1-2
```

### Modify Values

```python
matrix[0, 0] = 100
matrix[:, 1] = 0
matrix[0:2, 0:2] = 9
```

---

## ✅ Boolean Indexing

```python
arr = np.array([10, 25, 30, 45, 50])

mask = arr > 30
print(mask)       # [False False False  True  True]
print(arr[mask])  # [45 50]
```

### Common Filters

```python
arr[arr > 20]
arr[arr != 30]
arr[(arr > 20) & (arr < 50)]
arr[(arr < 20) | (arr > 40)]
```

> [!important] Use `&`, `|`, and `~`
> With NumPy arrays, use `&` instead of `and`, `|` instead of `or`, and wrap each condition in parentheses.

### Replace Values Conditionally

```python
scores = np.array([45, 78, 92, 55])
scores[scores < 60] = 0
```

---

## 🎲 Fancy Indexing

```python
arr = np.array([10, 20, 30, 40, 50])

arr[[0, 2, 4]]     # [10 30 50]
```

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

matrix[[0, 2]]        # rows 0 and 2
matrix[[0, 1], [1, 2]] # elements (0,1) and (1,2): [2 6]
```

---

## ➕ Vectorized Operations

```python
arr = np.array([1, 2, 3, 4])

arr + 10       # [11 12 13 14]
arr * 2        # [2 4 6 8]
arr ** 2       # [1 4 9 16]
arr / 2        # [0.5 1.  1.5 2. ]
```

### Array-to-Array Operations

```python
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

a + b          # [11 22 33]
a * b          # [10 40 90]
```

### Avoid This

```python
result = []
for value in arr:
    result.append(value * 2)
```

### Prefer This

```python
result = arr * 2
```

---

## 🧮 Universal Functions

```python
arr = np.array([1, 4, 9, 16])

np.sqrt(arr)
np.log(arr)
np.exp(arr)
np.sin(arr)
np.abs(np.array([-3, -2, 1]))
np.round(np.array([1.234, 5.678]), 2)
```

---

## 📊 Aggregations

```python
arr = np.array([10, 20, 30, 40])

arr.sum()
arr.mean()
arr.min()
arr.max()
arr.std()
arr.var()
arr.argmax()   # index of max value
arr.argmin()   # index of min value
```

### Axis-Based Aggregations

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

matrix.sum()          # 21
matrix.sum(axis=0)    # column sums: [5 7 9]
matrix.sum(axis=1)    # row sums: [6 15]
matrix.mean(axis=0)   # column means
matrix.mean(axis=1)   # row means
```

> [!tip] Axis Memory Trick
> `axis=0` collapses rows and returns one result per column.  
> `axis=1` collapses columns and returns one result per row.

---

## 📡 Broadcasting

Broadcasting lets NumPy operate on arrays with different but compatible shapes.

### Simple Cases

```python
arr = np.array([1, 2, 3])
arr + 10
```

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

matrix + np.array([10, 20, 30])
```

### Broadcasting Rules

Compare shapes from right to left:

1. Dimensions are compatible if they are equal.
2. Dimensions are compatible if one of them is `1`.
3. If neither condition is true, broadcasting fails.

### Shape Examples

| Shape A | Shape B | Works? | Result |
|--------|---------|--------|--------|
| `(3,)` | `()` | Yes | `(3,)` |
| `(2, 3)` | `(3,)` | Yes | `(2, 3)` |
| `(2, 3)` | `(2, 1)` | Yes | `(2, 3)` |
| `(2, 3)` | `(2,)` | No | Error |
| `(3, 1)` | `(1, 4)` | Yes | `(3, 4)` |

### Add New Axis

```python
arr = np.array([1, 2, 3])

arr[:, np.newaxis]    # shape (3, 1)
arr[np.newaxis, :]    # shape (1, 3)
```

---

## 🧼 Common Data Cleaning Patterns

### Normalize Values

```python
values = np.array([10, 20, 30, 40])
normalized = (values - values.mean()) / values.std()
```

### Min-Max Scale

```python
scaled = (values - values.min()) / (values.max() - values.min())
```

### Clip Outliers

```python
values = np.array([10, 20, 30, 1000])
clipped = np.clip(values, 0, 100)
```

### Handle Missing Values

```python
values = np.array([1.0, np.nan, 3.0])

np.isnan(values)
values[~np.isnan(values)]
np.nanmean(values)
np.nanmedian(values)
np.nanstd(values)
```

### Conditional Selection

```python
scores = np.array([45, 70, 88, 30])
labels = np.where(scores >= 60, "Pass", "Fail")
```

---

## 🔗 Combining Arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])
```

```python
x = np.array([[1, 2],
              [3, 4]])
y = np.array([[5, 6],
              [7, 8]])

np.vstack([x, y])       # stack rows
np.hstack([x, y])       # stack columns
np.concatenate([x, y], axis=0)
np.concatenate([x, y], axis=1)
```

---

## ✂️ Splitting Arrays

```python
arr = np.arange(10)

np.split(arr, 2)        # 2 equal parts
np.array_split(arr, 3)  # 3 near-equal parts
```

---

## 🔍 Sorting and Unique Values

```python
arr = np.array([3, 1, 2, 3, 2, 1])

np.sort(arr)
np.unique(arr)
np.unique(arr, return_counts=True)
```

### Argsort

```python
scores = np.array([70, 90, 80])
order = np.argsort(scores)

print(order)          # [0 2 1]
print(scores[order])  # [70 80 90]
```

---

## 🧱 Linear Algebra Basics

```python
a = np.array([[1, 2],
              [3, 4]])

b = np.array([[10, 20],
              [30, 40]])

a @ b                 # matrix multiplication
np.dot(a, b)          # also matrix multiplication
np.linalg.inv(a)      # inverse
np.linalg.det(a)      # determinant
```

> [!warning] `*` vs `@`
> `*` performs element-wise multiplication.  
> `@` performs matrix multiplication.

---

## 💾 Save and Load Arrays

```python
arr = np.array([1, 2, 3])

np.save("array.npy", arr)
loaded = np.load("array.npy")
```

```python
np.savetxt("array.csv", arr, delimiter=",")
np.loadtxt("array.csv", delimiter=",")
```

---

## 🧪 Mini Recipes

### Create a 3x3 Matrix from 1 to 9

```python
matrix = np.arange(1, 10).reshape(3, 3)
```

### Extract Even Numbers

```python
arr = np.arange(1, 11)
evens = arr[arr % 2 == 0]
```

### Replace Negative Values with Zero

```python
arr = np.array([-2, 5, -1, 8])
arr[arr < 0] = 0
```

### Standardize Each Column

```python
data = np.array([[10, 100],
                 [20, 200],
                 [30, 300]])

standardized = (data - data.mean(axis=0)) / data.std(axis=0)
```

### Normalize Each Row

```python
data = np.array([[1, 2, 3],
                 [4, 5, 6]])

row_sums = data.sum(axis=1, keepdims=True)
normalized = data / row_sums
```

### Convert Celsius to Fahrenheit

```python
celsius = np.array([0, 20, 37, 100])
fahrenheit = celsius * 9 / 5 + 32
```

---

## 🚨 Common Errors

### Shape Mismatch

```python
np.array([1, 2, 3]) + np.array([10, 20])
# ValueError: operands could not be broadcast together
```

Fix by checking shapes:

```python
print(a.shape)
print(b.shape)
```

### Using `and` / `or` with Arrays

```python
# Wrong
arr[(arr > 10) and (arr < 50)]

# Correct
arr[(arr > 10) & (arr < 50)]
```

### Forgetting Parentheses in Boolean Conditions

```python
# Wrong
arr[arr > 10 & arr < 50]

# Correct
arr[(arr > 10) & (arr < 50)]
```

### Accidentally Modifying a View

```python
arr = np.array([1, 2, 3, 4])
view = arr[1:3]
view[0] = 99

print(arr)  # [ 1 99  3  4]
```

Use `.copy()` when you need independent data:

```python
safe_copy = arr[1:3].copy()
```

---

## 🧾 Quick Command Table

| Task | Code |
|------|------|
| Import NumPy | `import numpy as np` |
| Create array | `np.array([1, 2, 3])` |
| Create zeros | `np.zeros((2, 3))` |
| Create ones | `np.ones((2, 3))` |
| Create range | `np.arange(0, 10, 2)` |
| Create evenly spaced values | `np.linspace(0, 1, 5)` |
| Random integers | `np.random.randint(1, 10, size=5)` |
| Shape | `arr.shape` |
| Dimensions | `arr.ndim` |
| Total elements | `arr.size` |
| Data type | `arr.dtype` |
| Reshape | `arr.reshape(3, 4)` |
| Flatten | `arr.flatten()` |
| Transpose | `arr.T` |
| Row slice | `matrix[0, :]` |
| Column slice | `matrix[:, 0]` |
| Boolean filter | `arr[arr > 10]` |
| Conditional values | `np.where(condition, a, b)` |
| Sum | `arr.sum()` |
| Mean | `arr.mean()` |
| Standard deviation | `arr.std()` |
| Min / max | `arr.min()`, `arr.max()` |
| Sort | `np.sort(arr)` |
| Unique values | `np.unique(arr)` |
| Concatenate | `np.concatenate([a, b])` |
| Matrix multiply | `a @ b` |
| Save array | `np.save("file.npy", arr)` |
| Load array | `np.load("file.npy")` |

---

## 🎤 Interview Quick Answers

**Why is NumPy faster than Python lists?**

> NumPy arrays store values in contiguous memory with a fixed data type, so operations can run in optimized compiled code instead of slow Python loops.

**What is vectorization?**

> Vectorization means applying operations to entire arrays at once instead of looping element by element in Python.

**What is broadcasting?**

> Broadcasting is NumPy's rule-based system for performing operations between arrays of different but compatible shapes.

**What is the difference between `arr.shape` and `arr.size`?**

> `shape` returns the dimensions of the array. `size` returns the total number of elements.

**What is the difference between a view and a copy?**

> A view shares memory with the original array. A copy owns independent memory. Modifying a view can modify the original.

**What does `axis=0` mean?**

> It means operate down the rows, producing one result per column.

**What does `axis=1` mean?**

> It means operate across columns, producing one result per row.

---

## ✅ Final Checklist

- [ ] I can create arrays using `array`, `zeros`, `ones`, `arange`, and `linspace`
- [ ] I can check `shape`, `ndim`, `size`, and `dtype`
- [ ] I can index and slice 1D and 2D arrays
- [ ] I can filter arrays using Boolean masks
- [ ] I can replace loops with vectorized operations
- [ ] I can use aggregations with `axis=0` and `axis=1`
- [ ] I can predict simple broadcasting behavior
- [ ] I can handle `np.nan` values
- [ ] I can reshape, flatten, stack, and split arrays

---

## 🔗 Navigation

| Previous | Back to Agenda |
|----------|----------------|
| [[06-numpy-exercises]] | [[00-agenda]] |

---

*Tags: #numpy #cheat-sheet #arrays #indexing #vectorization #broadcasting*
