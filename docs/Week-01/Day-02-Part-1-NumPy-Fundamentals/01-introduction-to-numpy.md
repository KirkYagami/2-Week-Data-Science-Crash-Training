# Introduction to NumPy
## The Foundation of Scientific Python

When you process a dataset with a million rows, you cannot afford to wait. The choice between a Python list and a NumPy array is the difference between a script that runs in 80 milliseconds and one that takes 8 seconds. That gap only widens as data grows. Understanding *why* NumPy is fast — not just *that* it is fast — makes you a better engineer because you stop guessing when to use it.

## Learning Objectives

- Explain the fundamental memory difference between Python lists and NumPy arrays
- Describe what the ndarray is and why it is designed that way
- Read and interpret the key array attributes: `shape`, `ndim`, `size`, `dtype`, `itemsize`, `nbytes`
- Choose the correct dtype for a use case and understand the consequence of getting it wrong
- Recognize where NumPy sits in the Python data science ecosystem

---

## The Problem With Python Lists

Python lists are general-purpose. They can hold a mix of integers, strings, objects, other lists — anything. That flexibility has a cost.

```python
import numpy as np

# A Python integer is a full object: type tag, reference count, value
# Each one consumes ~28 bytes on CPython
import sys
print(sys.getsizeof(42))        # Output: 28

# A list of 1 million integers
data = list(range(1_000_000))
# The list stores 1M pointers (8 bytes each) to 1M scattered objects
# Total: ~36 MB just for the list structure, plus the objects themselves
```

Three things make lists slow for numerical work:

**Scattered memory.** Each list element is a pointer to an object stored somewhere in the heap. When you iterate, the CPU constantly chases pointers to random memory locations. This destroys cache efficiency — the CPU prefetcher cannot predict where to look next.

**Type overhead per element.** Every Python integer carries metadata: a type pointer, a reference count, and the actual value. That is 28 bytes for a number that needs 8. For large arrays, this is a 3.5x memory waste.

**No SIMD.** Modern CPUs can perform the same operation on multiple values simultaneously (Single Instruction, Multiple Data). This only works on contiguous blocks of the same type. Python loops cannot use it.

---

## How NumPy Solves This

A NumPy array stores its data as a contiguous block of a single, fixed type:

```
Python List Memory Layout:
┌──────────────────────────────────────────────┐
│  [ptr] → [obj: type|refcount|28] scattered   │
│  [ptr] → [obj: type|refcount|28] scattered   │
│  [ptr] → [obj: type|refcount|28] scattered   │
└──────────────────────────────────────────────┘

NumPy Array Memory Layout:
┌─────────────────────────────────────────────┐
│  [8B][8B][8B][8B][8B][8B][8B][8B]...        │
│  ← contiguous float64 values in one block → │
└─────────────────────────────────────────────┘
```

The benefits cascade:

- The CPU's prefetcher loads the next values before you ask for them
- SIMD instructions process 4 or 8 values per clock cycle
- Computation happens in compiled C code, not interpreted Python bytecode
- Memory usage is 3–4x lower for typical numeric data

```python
import numpy as np
import time

# Build the same data as a list and as an array
data_list = list(range(1_000_000))
data_array = np.arange(1_000_000, dtype=np.float64)

# Python list: loop and square each element
start = time.perf_counter()
result_list = [x * x for x in data_list]
list_time = time.perf_counter() - start

# NumPy: vectorized squaring
start = time.perf_counter()
result_array = data_array ** 2
numpy_time = time.perf_counter() - start

print(f"List time:  {list_time * 1000:.1f} ms")
print(f"NumPy time: {numpy_time * 1000:.1f} ms")
print(f"Speedup:    {list_time / numpy_time:.0f}x")
# Output:
# List time:  82.3 ms
# NumPy time:  0.8 ms
# Speedup:    100x
```

> [!info] Why 100x?
> The exact speedup depends on what the operation is and the hardware. Embarrassingly parallel operations like element-wise math typically run 50–200x faster. Operations with data dependencies (e.g., iterative algorithms) see smaller gains. The speedup comes from eliminating Python interpreter overhead, not from magic.

---

## The ndarray — NumPy's Core Object

Every NumPy array is an instance of `numpy.ndarray`. You rarely construct one directly — you use creation functions — but understanding what it stores matters.

An ndarray has:
- A **data buffer**: the raw contiguous block of bytes
- A **dtype**: describes what each element is (float64, int32, bool, etc.)
- A **shape**: a tuple of integers giving the size along each dimension
- **Strides**: how many bytes to jump to move one step along each axis

Strides are the mechanism that makes slicing without copying possible. When you take a slice, NumPy creates a new array object pointing into the same buffer with adjusted strides — no data is moved.

```python
import numpy as np

# 0-D: a scalar wrapped in an array
scalar = np.array(42)
print(scalar.ndim)   # Output: 0
print(scalar.shape)  # Output: ()

# 1-D: a vector
vector = np.array([1.0, 2.0, 3.0, 4.0])
print(vector.ndim)   # Output: 1
print(vector.shape)  # Output: (4,)

# 2-D: a matrix (rows × columns)
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
print(matrix.ndim)   # Output: 2
print(matrix.shape)  # Output: (2, 3)

# 3-D: a tensor (think stack of matrices)
tensor = np.array([[[1, 2], [3, 4]],
                   [[5, 6], [7, 8]]])
print(tensor.ndim)   # Output: 3
print(tensor.shape)  # Output: (2, 2, 2)
```

### The Axis Convention

```
1-D array:   [a, b, c, d]
              ← axis 0 →

2-D array:
              ← axis 1 →
           ↑  [a, b, c]
    axis 0 |  [d, e, f]
           ↓  [g, h, i]

3-D array: shape (depth, rows, cols)
  axis 0 → which matrix in the stack (depth)
  axis 1 → which row within that matrix
  axis 2 → which column within that row
```

> [!tip] Reading Shapes
> For 2-D arrays, shape is always `(rows, columns)`. For higher dimensions, the axes go from outermost to innermost — the last two are always rows and columns.

---

## Key Array Attributes

```python
import numpy as np

arr = np.array([[1.0, 2.0, 3.0],
                [4.0, 5.0, 6.0]])

print(arr.ndim)     # Output: 2          — number of dimensions
print(arr.shape)    # Output: (2, 3)     — size along each axis
print(arr.size)     # Output: 6          — total number of elements
print(arr.dtype)    # Output: float64    — data type of each element
print(arr.itemsize) # Output: 8          — bytes per element
print(arr.nbytes)   # Output: 48         — total memory (size × itemsize)

# Strides tell you how many bytes to move per step along each axis
print(arr.strides)  # Output: (24, 8) — 24 bytes per row, 8 bytes per column
```

> [!info] Why `nbytes` Matters
> When your dataset has 10 million float64 values, `nbytes` tells you it takes 80 MB. If you switch to float32, that drops to 40 MB. For deep learning models with billions of parameters, this difference is everything.

---

## The dtype System

dtype is not cosmetic. It controls how much memory each element uses and what values it can represent. Getting dtype wrong creates two categories of bug: silent overflow (values wrap around silently) and unnecessary memory waste.

```python
import numpy as np

# Integer types — range grows with bit width
np.int8    # -128 to 127
np.int16   # -32,768 to 32,767
np.int32   # -2.1 billion to 2.1 billion
np.int64   # default on 64-bit systems, very large range

# Unsigned integers — useful when values cannot be negative
np.uint8   # 0 to 255  ← the standard for image pixels
np.uint16  # 0 to 65,535

# Float types — precision vs memory tradeoff
np.float16 # half precision, used in GPU training to save VRAM
np.float32 # single precision, standard in deep learning
np.float64 # double precision, NumPy default, highest precision

# Other
np.bool_   # True / False, 1 byte each
np.complex128  # complex numbers

# NumPy infers dtype from the input
arr_int   = np.array([1, 2, 3])
arr_float = np.array([1.0, 2.0, 3.0])
arr_bool  = np.array([True, False, True])

print(arr_int.dtype)   # Output: int64
print(arr_float.dtype) # Output: float64
print(arr_bool.dtype)  # Output: bool

# Specify dtype explicitly
arr_f32 = np.array([1, 2, 3], dtype=np.float32)
print(arr_f32.dtype)   # Output: float32

# Convert dtype with astype (returns a new array)
arr_f64 = arr_f32.astype(np.float64)
print(arr_f64.dtype)   # Output: float64
```

> [!warning] Integer Overflow Is Silent
> ```python
> # uint8 holds 0 to 255. What happens at 256?
> arr = np.array([254, 255], dtype=np.uint8)
> arr = arr + 2
> print(arr)  # Output: [0 1]  ← wrapped around silently!
>
> # This is a real bug in image processing code.
> # Always cast to int32 before arithmetic, then cast back.
> arr = np.array([200], dtype=np.uint8)
> result = arr.astype(np.int32) + 100
> result = result.clip(0, 255).astype(np.uint8)
> print(result)  # Output: [255]  ← correctly clamped
> ```

> [!warning] Mixed Types Cause Upcasting
> ```python
> arr = np.array([1, 2, 3])        # int64
> result = arr + 1.5               # float64, because int + float → float
> print(result.dtype)              # Output: float64
>
> # This is usually fine, but if you expected int output, check your dtypes.
> ```

---

## Memory Layout: C vs Fortran Order

This is advanced but worth knowing. NumPy stores 2-D arrays in **row-major order** (C order) by default: the elements of each row are contiguous in memory.

```
2-D array [[1, 2, 3],      C order (row-major):
            [4, 5, 6]]:    memory → [1, 2, 3, 4, 5, 6]

                           Fortran order (column-major):
                           memory → [1, 4, 2, 5, 3, 6]
```

```python
import numpy as np

arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.strides)         # Output: (24, 8) — row stride 24B, col stride 8B

arr_f = np.asfortranarray(arr)
print(arr_f.strides)       # Output: (8, 16) — col stride 8B, row stride 16B
```

> [!info] When Does This Matter?
> For most day-to-day work, it does not. It matters when you call BLAS/LAPACK routines (used by `np.linalg`) or pass arrays to external libraries that expect Fortran-order (some legacy scientific code). NumPy handles the conversion automatically in most cases.

---

## NumPy In the Ecosystem

NumPy does not work alone. It is the foundation that every major data science library builds on:

```
              ┌─────────────────────────────────────┐
              │       Your Analysis / Model         │
              └─────────────────────────────────────┘
                    ↓           ↓          ↓
         ┌──────────────┐ ┌─────────┐ ┌──────────────┐
         │    Pandas    │ │Matplotlib│ │ Scikit-learn │
         │  DataFrames  │ │  Plots  │ │  ML Models   │
         └──────────────┘ └─────────┘ └──────────────┘
                    ↓           ↓          ↓
         ┌──────────────────────────────────────────┐
         │                  NumPy                   │
         │           ndarray + math functions       │
         └──────────────────────────────────────────┘
                              ↓
         ┌──────────────────────────────────────────┐
         │         C / BLAS / LAPACK / SIMD         │
         └──────────────────────────────────────────┘
```

When Pandas says `.values`, it returns a NumPy array. When Scikit-learn fits a model, it works on NumPy arrays internally. When PyTorch moves data to CPU, it mirrors NumPy's interface. Learning NumPy well means you understand what all these libraries are actually doing with your data.

---

## Your First Meaningful Program

```python
import numpy as np

# Five students, three exams each
scores = np.array([
    [85, 92, 78],
    [90, 88, 95],
    [72, 65, 80],
    [88, 91, 87],
    [60, 70, 75],
])

print(f"Shape: {scores.shape}")          # Output: (5, 3)
print(f"dtype: {scores.dtype}")          # Output: int64
print(f"Memory: {scores.nbytes} bytes")  # Output: 120 bytes

# Average score per student (collapse across columns)
per_student = scores.mean(axis=1)
print(per_student.round(1))
# Output: [85.  91.  72.3 88.7 68.3]

# Average score per exam (collapse across rows)
per_exam = scores.mean(axis=0)
print(per_exam.round(1))
# Output: [79.  81.2 83. ]

# Who passed all three exams?
passing = np.all(scores >= 70, axis=1)
print(passing)
# Output: [ True  True False  True False]
```

> [!success] Key Takeaways
> - NumPy is fast because of contiguous memory, fixed dtype, and compiled C code
> - The ndarray has `shape`, `ndim`, `size`, `dtype`, `itemsize`, `nbytes` — know all six
> - dtype determines memory use and overflow behavior — set it explicitly when it matters
> - NumPy is the foundation every major data science library builds on
> - Axes go outermost to innermost; `axis=0` is rows, `axis=1` is columns

---

[[00-agenda]] | [[02-array-creation]]
