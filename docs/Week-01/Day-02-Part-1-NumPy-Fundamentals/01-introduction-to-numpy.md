# 01 — Introduction to NumPy
## The Foundation of Scientific Python

> [!quote] "NumPy is the reason Python became the language of data science."

---

## 🧭 Table of Contents

- [[#What is NumPy?]]
- [[#Why NumPy? The Problem with Python Lists]]
- [[#How NumPy Solves This]]
- [[#The ndarray — NumPy's Core Object]]
- [[#Key Terminology]]
- [[#Your First NumPy Program]]
- [[#NumPy in the Ecosystem]]
- [[#Summary]]

---

## What is NumPy?

**NumPy** stands for **Numerical Python**.

It is an open-source Python library that provides:
- A powerful **N-dimensional array object** (`ndarray`)
- **Mathematical functions** that operate on entire arrays at once
- Tools for **linear algebra**, **Fourier transforms**, and **random number generation**
- A foundation that **Pandas, Matplotlib, Scikit-learn, TensorFlow** all build upon

> [!info] Quick Fact
> NumPy was created by **Travis Oliphant** in 2005. It combined the best of two older libraries: `Numeric` and `Numarray`. Today it has hundreds of millions of downloads per month.

---

## Why NumPy? The Problem with Python Lists

### 🐢 Python Lists Are Slow for Math

Imagine you have 1 million numbers and want to double each one.

**Pure Python way:**

```python
# Create a list of 1 million numbers
data = list(range(1_000_000))

# Double each number
result = []
for x in data:
    result.append(x * 2)
```

This works, but it is **slow and verbose**. Why?

### 🔍 Under the Hood: Why Lists Are Slow

Python lists have a fundamental problem for numerical work:

```
Python List Memory Layout:
┌──────────────────────────────────────────────┐
│  [ptr1] [ptr2] [ptr3] [ptr4] [ptr5] ...      │
│    ↓       ↓      ↓      ↓      ↓            │
│  [obj]  [obj]  [obj]  [obj]  [obj]           │
│  type   type   type   type   type            │
│  value  value  value  value  value           │
│  ref    ref    ref    ref    ref             │
└──────────────────────────────────────────────┘
```

Each Python **integer** is actually a full Python **object** (28 bytes!). A list stores **pointers** to these objects scattered in memory.

**Problems:**
1. 🏃 **No vectorization** — Python loops one element at a time
2. 🧠 **Extra memory** — each number carries type info, reference count, etc.
3. 📍 **No contiguous memory** — objects scattered everywhere (bad for CPU cache)
4. 🔒 **Type flexibility** — Python must check types at runtime for every operation

### ⏱️ Benchmark: List vs NumPy

```python
import numpy as np
import time

# Python list approach
data_list = list(range(1_000_000))
start = time.time()
result_list = [x * 2 for x in data_list]
list_time = time.time() - start

# NumPy approach
data_array = np.arange(1_000_000)
start = time.time()
result_array = data_array * 2
numpy_time = time.time() - start

print(f"List time:  {list_time:.4f} seconds")
print(f"NumPy time: {numpy_time:.4f} seconds")
print(f"NumPy is {list_time / numpy_time:.0f}x faster!")

# Typical output:
# List time:  0.0821 seconds
# NumPy time: 0.0008 seconds
# NumPy is ~100x faster!
```

---

## How NumPy Solves This

NumPy arrays solve every problem Python lists have:

```
NumPy Array Memory Layout:
┌────────────────────────────────────────────────┐
│  [val1][val2][val3][val4][val5][val6]...        │
│  float64 · float64 · float64 · float64 ...     │
│  ← contiguous block of memory →                │
└────────────────────────────────────────────────┘
```

| Feature | Python List | NumPy Array |
|---------|------------|-------------|
| Memory layout | Scattered pointers | **Contiguous block** |
| Element type | Mixed (any object) | **Single fixed type** |
| Math operations | Element-by-element loop | **Vectorized (C speed)** |
| Memory per number | ~28 bytes | **8 bytes (float64)** |
| Speed for math | Slow | **10–1000x faster** |
| Multi-dimensional | Nested lists | **True N-D arrays** |

### 🚀 How NumPy Gets Its Speed

1. **Implemented in C** — the actual computation happens in compiled C code
2. **Contiguous memory** — CPU caches work efficiently on sequential data
3. **SIMD instructions** — modern CPUs process multiple numbers simultaneously
4. **No type checking** — all elements are the same type, decided upfront
5. **BLAS/LAPACK** — uses optimized math libraries under the hood

---

## The ndarray — NumPy's Core Object

Everything in NumPy revolves around the **`ndarray`** (N-Dimensional Array).

```python
import numpy as np

# Create your first ndarray
arr = np.array([1, 2, 3, 4, 5])
print(arr)        # [1 2 3 4 5]
print(type(arr))  # <class 'numpy.ndarray'>
```

### 📐 Dimensions (Axes)

NumPy arrays can have any number of dimensions:

```python
# 0-D array (scalar)
scalar = np.array(42)
print(scalar.ndim)   # 0
print(scalar.shape)  # ()

# 1-D array (vector)
vector = np.array([1, 2, 3, 4])
print(vector.ndim)   # 1
print(vector.shape)  # (4,)

# 2-D array (matrix)
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
print(matrix.ndim)   # 2
print(matrix.shape)  # (2, 3)  ← 2 rows, 3 columns

# 3-D array (tensor)
tensor = np.array([[[1, 2], [3, 4]],
                   [[5, 6], [7, 8]]])
print(tensor.ndim)   # 3
print(tensor.shape)  # (2, 2, 2)
```

### 🏷️ Essential Array Attributes

Every `ndarray` has these attributes you'll use constantly:

```python
arr = np.array([[1.0, 2.0, 3.0],
                [4.0, 5.0, 6.0]])

print(arr.ndim)    # 2           ← number of dimensions
print(arr.shape)   # (2, 3)      ← size of each dimension
print(arr.size)    # 6           ← total number of elements
print(arr.dtype)   # float64     ← data type of elements
print(arr.itemsize)# 8           ← bytes per element
print(arr.nbytes)  # 48          ← total bytes used (size × itemsize)
```

> [!tip] Shape is Always a Tuple
> `shape` returns `(rows, columns)` for 2D arrays. For 1D: `(n,)` — notice the trailing comma, that's how Python makes a 1-element tuple.

---

## Key Terminology

| Term | Meaning | Example |
|------|---------|---------|
| **ndarray** | NumPy's array object | `np.array([1,2,3])` |
| **axis** | A dimension of the array | axis=0 is rows, axis=1 is columns |
| **rank** | Number of dimensions (ndim) | A matrix has rank 2 |
| **shape** | Size along each dimension | `(3, 4)` = 3 rows, 4 cols |
| **dtype** | Data type of elements | `float64`, `int32`, `bool` |
| **scalar** | A 0-D array (single value) | `np.array(5)` |
| **vector** | A 1-D array | `np.array([1,2,3])` |
| **matrix** | A 2-D array | `np.array([[1,2],[3,4]])` |
| **tensor** | A 3-D or higher array | Used in deep learning |

### 📊 Visual: Axis Convention

```
1-D array:  [a, b, c, d]
             ←  axis 0  →

2-D array:
            ←  axis 1  →
          ↑  [a, b, c]
  axis 0  |  [d, e, f]
          ↓  [g, h, i]

3-D array: Think of it as a stack of matrices
  axis 0 → which matrix (depth)
  axis 1 → which row
  axis 2 → which column
```

---

## Data Types (dtype)

NumPy supports many numeric types. The most common:

```python
# Integer types
np.int8    # -128 to 127
np.int16   # -32,768 to 32,767
np.int32   # -2.1B to 2.1B
np.int64   # Very large integers (default on 64-bit systems)

# Unsigned integer types
np.uint8   # 0 to 255 (great for image pixels!)
np.uint32  # 0 to 4.3B

# Float types
np.float16 # Half precision (saves memory)
np.float32 # Single precision (common in deep learning)
np.float64 # Double precision (default, most precise)

# Other
np.bool_   # True / False
np.complex128  # Complex numbers

# Checking and setting dtype:
arr = np.array([1, 2, 3])
print(arr.dtype)  # int64 (on most 64-bit systems)

arr_float = np.array([1.0, 2.0, 3.0])
print(arr_float.dtype)  # float64

# Specify dtype explicitly
arr_int32 = np.array([1, 2, 3], dtype=np.int32)
print(arr_int32.dtype)  # int32

# Convert dtype
arr_as_float = arr.astype(np.float64)
print(arr_as_float.dtype)  # float64
```

> [!warning] Be Careful with dtypes!
> If you create `np.array([1, 2, 3])` it becomes `int64`. If you then try to store `3.14` in it, it gets truncated to `3`! Always use the right dtype from the start.

---

## Your First NumPy Program

Let's put it all together with a meaningful example:

```python
import numpy as np

# ─── Problem: Analyze student test scores ───

# 5 students, 3 tests each
scores = np.array([
    [85, 92, 78],    # Student 1
    [90, 88, 95],    # Student 2
    [72, 65, 80],    # Student 3
    [88, 91, 87],    # Student 4
    [60, 70, 75],    # Student 5
])

print("=== Student Score Analysis ===")
print(f"Shape: {scores.shape}")        # (5, 3)
print(f"Total elements: {scores.size}") # 15
print(f"Data type: {scores.dtype}")    # int64

# Average score per student (average across columns, axis=1)
student_averages = scores.mean(axis=1)
print(f"\nStudent averages: {student_averages}")
# [85. 91. 72.33 88.67 68.33]

# Average score per test (average across rows, axis=0)
test_averages = scores.mean(axis=0)
print(f"Test averages: {test_averages}")
# [79. 81.2 83.]

# Highest score in the whole dataset
print(f"\nHighest score: {scores.max()}")    # 95
print(f"Lowest score: {scores.min()}")      # 60
print(f"Overall average: {scores.mean():.2f}")  # 79.73

# Who passed (above 75)?
passed = scores > 75
print(f"\nPassed matrix:\n{passed}")
```

---

## NumPy in the Ecosystem

NumPy doesn't work alone — it's the **foundation** everything else is built on:

```
              ┌─────────────────────────────────────┐
              │           Your Code / App           │
              └─────────────────────────────────────┘
                    ↓           ↓          ↓
         ┌──────────────┐ ┌─────────┐ ┌──────────────┐
         │    Pandas    │ │Matplotlib│ │ Scikit-learn │
         │  DataFrames  │ │  Plots  │ │  ML Models   │
         └──────────────┘ └─────────┘ └──────────────┘
                    ↓           ↓          ↓
              ┌─────────────────────────────────────┐
              │              NumPy                  │
              │          (ndarray + math)           │
              └─────────────────────────────────────┘
                              ↓
              ┌─────────────────────────────────────┐
              │         C / BLAS / LAPACK           │
              │      (Low-level, fast math)         │
              └─────────────────────────────────────┘
```

---

## Summary

> [!success] Key Takeaways
> 
> 1. **NumPy = Numerical Python** — the foundation of scientific computing in Python
> 2. **Arrays beat lists** for numerical work: faster, less memory, cleaner code
> 3. **ndarray** is NumPy's core — an N-dimensional, homogeneous, fixed-type array
> 4. Key attributes: `shape`, `ndim`, `size`, `dtype`, `itemsize`, `nbytes`
> 5. **dtype matters** — float64 is default for decimals, int64 for integers
> 6. NumPy is the **bedrock** — Pandas, Matplotlib, and ML libraries all use it

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[00-agenda]] | [[02-array-creation]] |

---

*Tags: #numpy #python #datascience #arrays #ndarray #fundamentals*