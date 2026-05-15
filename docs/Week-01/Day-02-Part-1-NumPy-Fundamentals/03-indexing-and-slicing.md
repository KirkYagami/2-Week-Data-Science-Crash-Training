# Indexing and Slicing
## Accessing and Modifying Array Data

Indexing is how you talk to your data. Filtering a dataset to rows where a condition is true, extracting a column of features, replacing outlier values — all of it goes through indexing. NumPy gives you four distinct indexing mechanisms, each with different behavior. The most important thing to understand is not the syntax, but which operations give you a *view* of the original data and which give you an independent *copy*. Getting this wrong is one of the most common sources of silent bugs in data science code.

## Learning Objectives

- Access any element or region of 1-D, 2-D, and 3-D arrays using index and slice syntax
- Use negative indices and step slices fluently
- Filter arrays with boolean conditions using `&`, `|`, `~`
- Use `np.where()` to apply element-wise conditional logic
- Use fancy indexing to extract non-contiguous subsets
- Predict with confidence whether any given indexing operation returns a view or a copy, and use `.copy()` defensively

---

## Zero-Based Indexing and Negative Indices

NumPy uses zero-based indexing, and supports negative indices that count from the end.

```python
import numpy as np

arr = np.array([10, 20, 30, 40, 50])
#               0   1   2   3   4   ← positive indices
#              -5  -4  -3  -2  -1   ← negative indices

print(arr[0])   # Output: 10  ← first element
print(arr[2])   # Output: 30
print(arr[-1])  # Output: 50  ← last element (same as arr[4])
print(arr[-2])  # Output: 40
```

Negative indices make "last element" and "last N elements" natural to write without knowing the length.

---

## 1-D Slicing

Slice syntax: `arr[start : stop : step]`

- `start` is included (default: 0)
- `stop` is excluded (default: length of the array)
- `step` is the increment (default: 1)

```python
import numpy as np

arr = np.array([0, 10, 20, 30, 40, 50, 60, 70, 80, 90])
#              [0   1   2   3   4   5   6   7   8   9]  ← indices

print(arr[2:5])    # Output: [20 30 40]  ← indices 2, 3, 4 (not 5)
print(arr[:4])     # Output: [ 0 10 20 30]
print(arr[6:])     # Output: [60 70 80 90]
print(arr[::2])    # Output: [ 0 20 40 60 80]  ← every 2nd element
print(arr[1::2])   # Output: [10 30 50 70 90]  ← every 2nd, starting at 1
print(arr[::-1])   # Output: [90 80 70 60 50 40 30 20 10  0]  ← reversed
print(arr[-3:])    # Output: [70 80 90]  ← last 3 elements
print(arr[:-3])    # Output: [ 0 10 20 30 40 50 60]
```

**Slice visualization:**

```
arr = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]

arr[2:7]:
          ┌────────────────────┐
[0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
           ↑                  ↑
         start=2           stop=7 (excluded)
Result: [20, 30, 40, 50, 60]

arr[::2]:
[0,  10, 20,  30, 40,  50, 60,  70, 80,  90]
 ↑        ↑        ↑        ↑        ↑
Result: [0, 20, 40, 60, 80]
```

---

## 2-D Indexing and Slicing

For a 2-D array (matrix), indexing uses **two positions**: `arr[row, col]`.

```python
import numpy as np

matrix = np.array([[11, 12, 13, 14],
                   [21, 22, 23, 24],
                   [31, 32, 33, 34]])
#                  col0 col1 col2 col3

# Single element: [row, col]
print(matrix[0, 0])    # Output: 11  ← row 0, column 0
print(matrix[1, 2])    # Output: 23  ← row 1, column 2
print(matrix[-1, -1])  # Output: 34  ← last row, last column

# Entire row
print(matrix[1])       # Output: [21 22 23 24]
print(matrix[1, :])    # Output: [21 22 23 24]  ← explicit slice, same result

# Entire column
print(matrix[:, 0])    # Output: [11 21 31]  ← all rows, column 0
print(matrix[:, 2])    # Output: [13 23 33]
```

**2-D slice mental model:** `matrix[row_slice, col_slice]`

```python
matrix = np.array([[11, 12, 13, 14],
                   [21, 22, 23, 24],
                   [31, 32, 33, 34],
                   [41, 42, 43, 44]])

# Top-left 2×2 block
print(matrix[:2, :2])
# Output:
# [[11 12]
#  [21 22]]

# Bottom-right 2×2 block
print(matrix[2:, 2:])
# Output:
# [[33 34]
#  [43 44]]

# Every other column
print(matrix[:, ::2])
# Output:
# [[11 13]
#  [21 23]
#  [31 33]
#  [41 43]]

# Reverse row order
print(matrix[::-1, :])
# Output:
# [[41 42 43 44]
#  [31 32 33 34]
#  [21 22 23 24]
#  [11 12 13 14]]
```

---

## 3-D Indexing

Think of a 3-D array as a stack of matrices. Index it with three coordinates: `[layer, row, col]`.

```python
import numpy as np

cube = np.array([[[1,  2,  3],
                  [4,  5,  6],
                  [7,  8,  9]],
                 [[10, 11, 12],
                  [13, 14, 15],
                  [16, 17, 18]]])

print(cube.shape)      # Output: (2, 3, 3) — 2 layers, 3 rows, 3 cols

print(cube[0, 0, 0])   # Output: 1   ← layer 0, row 0, col 0
print(cube[1, 2, 2])   # Output: 18  ← layer 1, row 2, col 2
print(cube[0, :, 1])   # Output: [2 5 8]  ← layer 0, all rows, col 1
print(cube[:, 1, :])   # Output: [[ 4  5  6], [13 14 15]]  ← row 1 of each layer
```

---

## Boolean Indexing

Boolean indexing is how you filter arrays. You create a boolean mask — an array of `True`/`False` values — and use it to select elements. It is the NumPy equivalent of a SQL `WHERE` clause, and it is used constantly.

```python
import numpy as np

scores = np.array([82, 47, 91, 35, 73, 68, 55, 88, 42, 76])

# A comparison produces a boolean array
mask = scores >= 70
print(mask)
# Output: [ True False  True False  True False False  True False  True]

# Use the mask to select elements
passing = scores[mask]
print(passing)  # Output: [82 91 73 88 76]

# More common: combine in one expression
print(scores[scores >= 70])  # Output: [82 91 73 88 76]

# Compound conditions — use & (and), | (or), ~ (not)
# IMPORTANT: use parentheses around each condition
print(scores[(scores >= 60) & (scores < 80)])  # Output: [82 73 68 76]  ← wait, 82 is not < 80
# Let me be precise:
print(scores[(scores > 60) & (scores < 80)])   # Output: [73 68 76]
print(scores[(scores < 50) | (scores > 85)])   # Output: [47 35 91 88 42]
print(scores[~(scores >= 70)])                 # Output: [47 35 68 55 42]
```

> [!warning] Never Use `and`, `or`, `not` With NumPy Arrays
> ```python
> arr = np.array([1, 2, 3, 4, 5])
>
> # This raises "ValueError: The truth value of an array is ambiguous"
> arr[(arr > 2) and (arr < 5)]   # ← WRONG
>
> # This silently gives wrong results due to operator precedence
> arr[arr > 2 & arr < 5]         # ← WRONG (& binds tighter than >)
>
> # Always use & | ~ with parentheses around each condition
> arr[(arr > 2) & (arr < 5)]     # ← Correct
> ```

### Boolean Indexing on 2-D Arrays

```python
import numpy as np

# Student records: [id, score]
students = np.array([[1, 82],
                     [2, 47],
                     [3, 91],
                     [4, 35],
                     [5, 73]])

# Select entire rows where score >= 70
# students[:, 1] extracts the score column
passed = students[students[:, 1] >= 70]
print(passed)
# Output:
# [[ 1 82]
#  [ 3 91]
#  [ 5 73]]

# Modify values in-place using boolean mask
matrix = np.array([[3, -1, 4],
                   [-2, 5, -3],
                   [7, -4, 2]])
matrix[matrix < 0] = 0   # Replace negatives with zero
print(matrix)
# Output:
# [[3 0 4]
#  [0 5 0]
#  [7 0 2]]
```

---

## `np.where()` — Element-wise If-Else

`np.where(condition, value_if_true, value_if_false)` applies a condition to every element and produces a new array based on the result.

```python
import numpy as np

arr = np.array([10, -5, 3, -8, 7, -2])

# Replace negatives with 0
result = np.where(arr > 0, arr, 0)
print(result)  # Output: [10  0  3  0  7  0]

# Assign labels
grades = np.array([85, 42, 91, 55, 73, 38])
labels = np.where(grades >= 60, 'Pass', 'Fail')
print(labels)  # Output: ['Pass' 'Fail' 'Pass' 'Fail' 'Pass' 'Fail']

# Nested np.where for multiple categories
letter = np.where(grades >= 90, 'A',
         np.where(grades >= 80, 'B',
         np.where(grades >= 70, 'C',
         np.where(grades >= 60, 'D', 'F'))))
print(letter)  # Output: ['B' 'F' 'A' 'F' 'C' 'F']

# np.where with no second/third argument returns indices
above_75 = np.where(grades > 75)
print(above_75)   # Output: (array([0, 2, 4]),)  ← tuple of index arrays
print(grades[above_75])  # Output: [85 91 73]  ← wait, 73 > 75 is False
# Let me correct:
above_70 = np.where(grades > 70)
print(grades[above_70])  # Output: [85 91 73]
```

> [!tip] `np.where` with One Argument Returns Indices
> `np.where(condition)` returns a tuple of arrays (one per dimension) with the indices where the condition is True. For 1-D: `np.where(arr > 5)[0]` gives the integer indices directly.

---

## Fancy Indexing

Fancy indexing means using an **array of indices** to pick elements. The result is always a **copy** — never a view.

```python
import numpy as np

arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

# Index with a list of positions
print(arr[[0, 3, 7]])    # Output: [10 40 80]

# Indices can repeat
print(arr[[0, 0, 3, 3]])  # Output: [10 10 40 40]

# Indices in a custom order
print(arr[[8, 4, 1]])     # Output: [90 50 20]
```

### Fancy Indexing on 2-D Arrays

```python
import numpy as np

matrix = np.array([[11, 12, 13],
                   [21, 22, 23],
                   [31, 32, 33],
                   [41, 42, 43]])

# Select specific rows
print(matrix[[0, 2], :])
# Output:
# [[11 12 13]
#  [31 32 33]]

# Paired row-column indexing — picks individual elements, not a submatrix
rows = [0, 1, 2]
cols = [0, 1, 2]
print(matrix[rows, cols])  # Output: [11 22 33]  ← the diagonal

# To get a submatrix with fancy indexing, use np.ix_
print(matrix[np.ix_([0, 2], [1, 2])])
# Output:
# [[12 13]
#  [32 33]]
```

> [!info] Fancy Indexing Always Returns a Copy
> This is different from basic slicing. You can safely modify the result without touching the original.
> ```python
> arr = np.array([10, 20, 30, 40, 50])
> result = arr[[1, 3]]   # fancy indexing → copy
> result[0] = 999
> print(arr)  # Output: [10 20 30 40 50]  ← unchanged
> ```

---

## Views vs Copies — The Silent Bug Factory

This is the most dangerous aspect of NumPy for beginners, because the bugs it creates are silent: the code runs without errors, but produces wrong results.

**The rule:** basic slicing returns a **view**. The view shares the underlying data buffer with the original. Modifying the view modifies the original.

```python
import numpy as np

original = np.array([1, 2, 3, 4, 5])

# Slicing creates a VIEW — same data, different array object
view = original[1:4]
print(view)  # Output: [2 3 4]

view[0] = 999
print(view)     # Output: [999   3   4]
print(original) # Output: [  1 999   3   4   5]
#                                    ^^^
#                      THE ORIGINAL CHANGED. This surprises most people.
```

This behavior is a feature, not a bug. NumPy avoids copying data on every slice, which is essential for performance. But if you do not know this, you create bugs that take hours to find.

### Full View vs Copy Reference

| Operation | Returns | Modifying it changes original? |
|-----------|---------|-------------------------------|
| `arr[1:4]` | View | Yes |
| `arr[::2]` | View | Yes |
| `arr.reshape(2, 3)` | View (usually) | Yes |
| `arr.ravel()` | View (usually) | Yes |
| `arr.T` | View | Yes |
| `arr[[1, 2, 3]]` | Copy | No |
| `arr[arr > 0]` | Copy | No |
| `arr.flatten()` | Copy | No |
| `arr.copy()` | Copy | No |

### How to Check If You Have a View

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])

view = arr[1:4]
copy = arr[1:4].copy()

# .base is the original array if this is a view, None if it's a copy
print(view.base is arr)   # Output: True  ← it's a view
print(copy.base is None)  # Output: True  ← it's a copy
```

### How to Force a Copy

```python
import numpy as np

original = np.array([1, 2, 3, 4, 5])

# .copy() creates an independent array
safe = original[1:4].copy()
safe[0] = 999
print(safe)     # Output: [999   3   4]
print(original) # Output: [1 2 3 4 5]  ← untouched
```

> [!warning] The Most Common View Bug in Practice
> ```python
> # You have a dataset and want to work on a subset
> dataset = np.random.randn(1000, 10)
> subset = dataset[:100, :]   # This is a VIEW
>
> # You normalize the subset
> subset = (subset - subset.mean()) / subset.std()
> # Wait — this reassigns the variable 'subset' to a new array (the result
> # of the arithmetic), so the original is safe here.
> # But watch this:
>
> subset = dataset[:100, :]
> subset -= subset.mean()     # In-place operation on a view!
> # Now dataset[:100, :] has been modified too!
> # The fix:
> subset = dataset[:100, :].copy()
> subset -= subset.mean()     # Safe — subset is independent
> ```

> [!tip] When in Doubt, Copy
> If you are not 100% sure whether something is a view or copy, call `.copy()`. The memory overhead is the cost of one extra allocation. The time you save not debugging a silent mutation bug is worth far more.

---

## Modifying Arrays via Indexing

Any indexing operation on the left side of an assignment modifies the array in-place.

```python
import numpy as np

arr = np.zeros(10, dtype=int)

arr[3] = 99
print(arr)  # Output: [0 0 0 99 0 0 0 0 0 0]

arr[5:8] = 7         # broadcast a scalar into a slice
print(arr)  # Output: [0 0 0 99 0 7 7 7 0 0]

arr[arr == 0] = -1   # boolean mask assignment
print(arr)  # Output: [-1 -1 -1 99 -1 7 7 7 -1 -1]

arr[[0, 2, 4]] = [100, 200, 300]   # fancy index assignment
print(arr)  # Output: [100 -1 200 99 300 7 7 7 -1 -1]
```

---

> [!success] Key Takeaways
> - Indexing uses `arr[i]` for 1-D, `arr[i, j]` for 2-D — always comma-separated, never `arr[i][j]`
> - Slices are `[start:stop:step]` — stop is excluded, step defaults to 1
> - Boolean indexing uses `&`, `|`, `~` — never `and`, `or`, `not` — with parentheses around each condition
> - `np.where(cond, a, b)` is element-wise if-else; `np.where(cond)` returns indices
> - Fancy indexing with integer arrays always returns a copy
> - Basic slicing always returns a view — modifying it modifies the original
> - Use `.copy()` whenever you need to modify a subset without affecting the source

---

[[02-array-creation]] | [[04-vectorization]]
