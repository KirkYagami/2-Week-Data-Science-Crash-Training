# NumPy Exercises
## Practice Problems — Warm-Up, Main, Stretch

> [!tip] How to Get the Most From These Exercises
> Open a Jupyter notebook or Python file. Read the problem. Close these notes and work through it yourself. Only check the solution when you are genuinely stuck or want to compare approaches. A solution you struggled toward is worth ten you read cold. If your solution works but looks different from the provided one, understand *why* — there is usually a reason one approach is preferred.

---

## Warm-Up — Array Basics

These confirm that you have the fundamentals and can navigate the API without hesitation.

### Exercise W1 — Create and Inspect

Create a 1-D array of even numbers from 2 to 30 inclusive. Print its shape, size, dtype, sum, and mean.

Expected output:
```
Array:  [ 2  4  6  8 10 12 14 16 18 20 22 24 26 28 30]
Shape:  (15,)
Size:   15
dtype:  int64
Sum:    240
Mean:   16.0
```

??? "Show answer"
    ```python
    import numpy as np

    arr = np.arange(2, 32, 2)
    print("Array: ", arr)
    print("Shape: ", arr.shape)
    print("Size:  ", arr.size)
    print("dtype: ", arr.dtype)
    print("Sum:   ", arr.sum())
    print("Mean:  ", arr.mean())
    ```

---

### Exercise W2 — Border Matrix

Create a 6×6 integer matrix where border elements are 1 and interior elements are 0. Do not use a loop.

Expected output:
```
[[1 1 1 1 1 1]
 [1 0 0 0 0 1]
 [1 0 0 0 0 1]
 [1 0 0 0 0 1]
 [1 0 0 0 0 1]
 [1 1 1 1 1 1]]
```

??? "Show answer"
    ```python
    import numpy as np

    mat = np.ones((6, 6), dtype=int)
    mat[1:-1, 1:-1] = 0
    print(mat)

    # Alternative starting from zeros:
    mat2 = np.zeros((6, 6), dtype=int)
    mat2[0, :]  = 1
    mat2[-1, :] = 1
    mat2[:, 0]  = 1
    mat2[:, -1] = 1
    print(mat2)
    ```

---

### Exercise W3 — Checkerboard

Create an 8×8 integer matrix with a checkerboard pattern of 0s and 1s. `(0, 0)` should be 0. Do not use a loop.

Expected output (partial):
```
[[0 1 0 1 0 1 0 1]
 [1 0 1 0 1 0 1 0]
 [0 1 0 1 0 1 0 1]
 ...]
```

??? "Show answer"
    ```python
    import numpy as np

    # Method 1: index parity
    board = np.zeros((8, 8), dtype=int)
    board[1::2, ::2]  = 1   # odd rows, even cols
    board[::2,  1::2] = 1   # even rows, odd cols
    print(board)

    # Method 2: sum of indices modulo 2
    rows, cols = np.indices((8, 8))
    board2 = (rows + cols) % 2
    print(board2)
    ```

---

### Exercise W4 — Reshape and Navigate 3-D

Create an array containing integers 1 through 60. Reshape it to shape `(3, 4, 5)`. Then:
1. Print the element at position `[2, 3, 4]`
2. Print the entire second "layer" (index 1 along axis 0)
3. Print all values greater than 45

??? "Show answer"
    ```python
    import numpy as np

    arr = np.arange(1, 61).reshape(3, 4, 5)

    # 1. Single element
    print(arr[2, 3, 4])   # Output: 60  ← last element

    # 2. Second layer (axis 0, index 1)
    print(arr[1])
    # Output:
    # [[21 22 23 24 25]
    #  [26 27 28 29 30]
    #  [31 32 33 34 35]
    #  [36 37 38 39 40]]

    # 3. Values greater than 45
    print(arr[arr > 45])
    # Output: [46 47 48 49 50 51 52 53 54 55 56 57 58 59 60]
    ```

---

## Main — Indexing, Vectorization, Broadcasting

Realistic problems requiring you to combine multiple concepts.

### Exercise M1 — Slicing Patterns

Given `arr = np.arange(20)`:

1. Extract every third element starting from index 1
2. Extract the last 6 elements in reverse order
3. Replace all odd-indexed elements with their negative (in-place)

Expected outputs:
```
1: [ 1  4  7 10 13 16 19]
2: [19 18 17 16 15 14]
3: [ 0 -1  2 -3  4 -5  6 -7  8 -9 10 -11 12 -13 14 -15 16 -17 18 -19]
```

??? "Show answer"
    ```python
    import numpy as np

    arr = np.arange(20)

    # 1. Every third, starting at index 1
    print(arr[1::3])   # Output: [ 1  4  7 10 13 16 19]

    # 2. Last 6 in reverse
    print(arr[-1:-7:-1])  # Output: [19 18 17 16 15 14]

    # 3. Negate odd-indexed elements in-place
    arr = np.arange(20)  # reset
    arr[1::2] = -arr[1::2]
    print(arr)
    # Output: [ 0 -1  2 -3  4 -5  6 -7  8 -9 10 -11 12 -13 14 -15 16 -17 18 -19]
    ```

---

### Exercise M2 — Boolean Filtering on a Matrix

You have salary data for employees across three departments.

```python
import numpy as np
rng = np.random.default_rng(42)
# Shape: (20, 3) — 20 employees, columns: [dept_id, years_exp, salary]
data = np.column_stack([
    rng.integers(0, 3, 20),          # dept: 0, 1, or 2
    rng.integers(1, 15, 20),         # years experience
    rng.integers(40000, 120000, 20)  # salary
])
```

Without using any loop:
1. Find all employees in department 1
2. Find all employees with salary > 80,000 AND experience > 5 years
3. Count how many employees earn above the overall mean salary
4. Print the salary of the highest-paid employee in department 2

??? "Show answer"
    ```python
    import numpy as np
    rng = np.random.default_rng(42)
    data = np.column_stack([
        rng.integers(0, 3, 20),
        rng.integers(1, 15, 20),
        rng.integers(40000, 120000, 20)
    ])

    # 1. Department 1 employees
    dept1 = data[data[:, 0] == 1]
    print(f"Dept 1 employees: {len(dept1)}")

    # 2. High salary AND experienced
    mask = (data[:, 2] > 80000) & (data[:, 1] > 5)
    high_earners = data[mask]
    print(f"High salary + experienced: {len(high_earners)}")

    # 3. Count above mean salary
    mean_salary = data[:, 2].mean()
    count_above = np.sum(data[:, 2] > mean_salary)
    print(f"Mean salary: {mean_salary:.0f}, Above mean: {count_above}")

    # 4. Top salary in department 2
    dept2_salaries = data[data[:, 0] == 2, 2]
    print(f"Highest salary in dept 2: {dept2_salaries.max()}")
    ```

---

### Exercise M3 — Vectorize This

Rewrite the following loop as a single vectorized NumPy expression. Verify the results match.

```python
import math

data = [3.5, 7.2, 1.1, 8.8, 4.5, 6.3, 2.9, 9.1, 5.0, 3.7]
mean = sum(data) / len(data)
std  = math.sqrt(sum((x - mean)**2 for x in data) / len(data))

result = []
for x in data:
    if x > mean:
        result.append(math.log(x / mean))
    else:
        result.append(-(mean - x) / std)
```

??? "Show answer"
    ```python
    import numpy as np

    data = np.array([3.5, 7.2, 1.1, 8.8, 4.5, 6.3, 2.9, 9.1, 5.0, 3.7])
    mean = data.mean()
    std  = data.std()

    # np.where handles the conditional branch vectorized
    result = np.where(
        data > mean,
        np.log(data / mean),
        -(mean - data) / std
    )
    print(result.round(4))
    # Both approaches produce the same output.
    ```

---

### Exercise M4 — Weekly Temperature Analysis

```python
import numpy as np
rng = np.random.default_rng(0)
# 4 weeks × 7 days × 24 hours of temperature readings (Celsius)
temps = 20 + rng.standard_normal((4, 7, 24)) * 6
# Shape: (4, 7, 24)
```

Using only NumPy (no loops):

1. What is the average temperature for each week? (one number per week)
2. Which hour of the day is coldest on average across all weeks and days?
3. What fraction of all readings exceeded 30°C?
4. What is the daily temperature range (max - min) for each day of the first week?
5. Identify the `(week, day)` combination with the highest single-hour reading

??? "Show answer"
    ```python
    import numpy as np
    rng = np.random.default_rng(0)
    temps = 20 + rng.standard_normal((4, 7, 24)) * 6

    # 1. Weekly averages: collapse days and hours
    weekly_avg = temps.mean(axis=(1, 2))
    print("Weekly averages:", weekly_avg.round(2))

    # 2. Coldest hour: average over weeks and days, then find argmin
    hourly_avg = temps.mean(axis=(0, 1))   # shape: (24,)
    coldest_hour = np.argmin(hourly_avg)
    print(f"Coldest hour: {coldest_hour} ({hourly_avg[coldest_hour]:.2f}°C)")

    # 3. Fraction above 30°C
    fraction_hot = np.mean(temps > 30)
    print(f"Fraction > 30°C: {fraction_hot:.3f} ({fraction_hot*100:.1f}%)")

    # 4. Daily range for week 0: max and min across hours (axis=2)
    week0 = temps[0]          # shape: (7, 24)
    daily_range = week0.max(axis=1) - week0.min(axis=1)
    print("Daily range (week 0):", daily_range.round(2))

    # 5. Week and day of highest single reading
    # Collapse hours: find max per (week, day)
    daily_max = temps.max(axis=2)    # shape: (4, 7)
    flat_idx = np.argmax(daily_max)  # index in flattened array
    week_idx, day_idx = np.unravel_index(flat_idx, daily_max.shape)
    peak_temp = daily_max[week_idx, day_idx]
    print(f"Peak reading: week {week_idx}, day {day_idx}, temp={peak_temp:.2f}°C")
    ```

---

### Exercise M5 — Views vs Copies Detective

For each operation below, predict whether the result is a view or a copy. Then verify using `.base`.

```python
import numpy as np

arr = np.arange(24).reshape(4, 6)

a = arr[1:3, :]        # prediction: ?
b = arr[[0, 2], :]     # prediction: ?
c = arr[arr > 10]      # prediction: ?
d = arr.T              # prediction: ?
e = arr.flatten()      # prediction: ?
f = arr.ravel()        # prediction: ?
g = arr.reshape(6, 4)  # prediction: ?
```

??? "Show answer"
    ```python
    import numpy as np

    arr = np.arange(24).reshape(4, 6)

    a = arr[1:3, :]
    b = arr[[0, 2], :]
    c = arr[arr > 10]
    d = arr.T
    e = arr.flatten()
    f = arr.ravel()
    g = arr.reshape(6, 4)

    results = {
        'a = arr[1:3,:]   (basic slice)': a,
        'b = arr[[0,2],:] (fancy index)': b,
        'c = arr[arr>10]  (bool index) ': c,
        'd = arr.T        (transpose)  ': d,
        'e = arr.flatten()(flatten)    ': e,
        'f = arr.ravel()  (ravel)      ': f,
        'g = arr.reshape  (reshape)    ': g,
    }

    for name, x in results.items():
        kind = 'VIEW' if x.base is not None else 'COPY'
        print(f"{name} → {kind}")

    # Expected output:
    # a = arr[1:3,:]    → VIEW
    # b = arr[[0,2],:]  → COPY
    # c = arr[arr>10]   → COPY
    # d = arr.T         → VIEW
    # e = arr.flatten() → COPY
    # f = arr.ravel()   → VIEW  (usually — depends on memory layout)
    # g = arr.reshape() → VIEW  (usually)
    ```

---

## Stretch — Harder, No Hints

These problems require combining multiple ideas. The expected output is shown; the path is yours to find.

### Exercise S1 — Implement Euclidean Distance Without `np.linalg`

Write a function `pairwise_distances(X)` that takes a 2-D array of shape `(n, d)` (n points in d-dimensional space) and returns the `(n, n)` pairwise distance matrix. No loops, no `np.linalg.norm` on pairs.

```python
import numpy as np
rng = np.random.default_rng(7)
X = rng.standard_normal((5, 3))
D = pairwise_distances(X)
# D[i, j] = Euclidean distance between point i and point j
# D should be symmetric, and D[i, i] = 0
```

Verify:
- `D` is symmetric: `np.allclose(D, D.T)` → True
- Diagonal is zero: `np.allclose(np.diag(D), 0)` → True

??? "Show answer"
    ```python
    import numpy as np

    def pairwise_distances(X):
        # X: (n, d)
        # Expand to (n, 1, d) and (1, n, d), then broadcast
        diff = X[:, np.newaxis, :] - X[np.newaxis, :, :]  # (n, n, d)
        return np.sqrt((diff ** 2).sum(axis=-1))           # (n, n)

    rng = np.random.default_rng(7)
    X = rng.standard_normal((5, 3))
    D = pairwise_distances(X)

    print("Symmetric:", np.allclose(D, D.T))       # True
    print("Zero diag:", np.allclose(np.diag(D), 0)) # True
    print(D.round(3))
    ```

---

### Exercise S2 — Complete Preprocessing Pipeline

```python
import numpy as np
rng = np.random.default_rng(99)
raw = rng.standard_normal((200, 6)) * np.array([10, 2, 50, 0.5, 100, 1])
# Introduce some outliers
raw[::20] *= 5
```

Build a pipeline (no loops) that:

1. Removes any row where at least one feature is more than 4 standard deviations from that feature's mean
2. Applies min-max normalization to each feature (scale to [0, 1])
3. Computes the `(6, 6)` correlation matrix of the normalized features using only NumPy
4. Finds the pair of features with the highest absolute correlation (excluding self-correlation)

??? "Show answer"
    ```python
    import numpy as np
    rng = np.random.default_rng(99)
    raw = rng.standard_normal((200, 6)) * np.array([10, 2, 50, 0.5, 100, 1])
    raw[::20] *= 5

    # Step 1: Remove outlier rows
    mean = raw.mean(axis=0)          # shape: (6,)
    std  = raw.std(axis=0)           # shape: (6,)
    z_scores = np.abs((raw - mean) / std)   # shape: (200, 6)
    clean_mask = np.all(z_scores <= 4, axis=1)
    clean = raw[clean_mask]
    print(f"Rows after outlier removal: {clean.shape[0]}")

    # Step 2: Min-max normalize each feature
    col_min = clean.min(axis=0)      # shape: (6,)
    col_max = clean.max(axis=0)      # shape: (6,)
    normalized = (clean - col_min) / (col_max - col_min)

    # Step 3: Correlation matrix using NumPy
    # np.corrcoef expects (features, samples) — transpose first
    corr = np.corrcoef(normalized.T)   # shape: (6, 6)
    print("Correlation matrix:")
    print(corr.round(3))

    # Step 4: Find highest off-diagonal absolute correlation
    # Mask out the diagonal (which is always 1)
    mask = ~np.eye(6, dtype=bool)
    abs_corr = np.abs(corr)
    abs_corr_masked = abs_corr * mask   # zero out diagonal

    flat_idx = np.argmax(abs_corr_masked)
    i, j = np.unravel_index(flat_idx, corr.shape)
    print(f"Highest correlation: features {i} and {j}, r = {corr[i, j]:.4f}")
    ```

---

### Exercise S3 — Rolling Window Statistics

Compute the rolling mean and rolling standard deviation for a time series using only NumPy (no Pandas, no loops for the window computation).

```python
import numpy as np
rng = np.random.default_rng(42)
prices = 100 + np.cumsum(rng.standard_normal(500) * 2)
window = 20
```

Expected behavior:
- `rolling_mean[i]` = mean of `prices[i : i + window]` for i in range(len(prices) - window + 1)
- Result shape: `(len(prices) - window + 1,)` = `(481,)`

??? "Show answer"
    ```python
    import numpy as np
    rng = np.random.default_rng(42)
    prices = 100 + np.cumsum(rng.standard_normal(500) * 2)
    window = 20
    n = len(prices) - window + 1   # 481

    # Build a (n, window) array using stride tricks
    # Each row is a sliding window view
    from numpy.lib.stride_tricks import as_strided

    item_size = prices.strides[0]
    windows = as_strided(
        prices,
        shape=(n, window),
        strides=(item_size, item_size)
    )
    # windows[i] = prices[i : i + window]

    rolling_mean = windows.mean(axis=1)
    rolling_std  = windows.std(axis=1)

    print(f"Shape: {rolling_mean.shape}")   # (481,)
    print(f"First rolling mean: {rolling_mean[0]:.3f}")
    print(f"Last rolling mean:  {rolling_mean[-1]:.3f}")

    # Verify first value
    expected_first = prices[:window].mean()
    print(f"Manual check: {expected_first:.3f}")  # should match rolling_mean[0]

    # Alternative without stride tricks (using broadcasting index array)
    indices = np.arange(window) + np.arange(n)[:, np.newaxis]  # shape: (n, window)
    rolling_mean_v2 = prices[indices].mean(axis=1)
    print("Methods agree:", np.allclose(rolling_mean, rolling_mean_v2))
    ```

---

## Progress Tracker

| Exercise | Solved Without Help? | Understood Solution? |
|----------|---------------------|---------------------|
| W1 — Create and Inspect | ☐ | ☐ |
| W2 — Border Matrix | ☐ | ☐ |
| W3 — Checkerboard | ☐ | ☐ |
| W4 — Reshape 3-D | ☐ | ☐ |
| M1 — Slicing Patterns | ☐ | ☐ |
| M2 — Boolean Filtering | ☐ | ☐ |
| M3 — Vectorize This | ☐ | ☐ |
| M4 — Temperature Analysis | ☐ | ☐ |
| M5 — Views vs Copies | ☐ | ☐ |
| S1 — Pairwise Distance | ☐ | ☐ |
| S2 — Preprocessing Pipeline | ☐ | ☐ |
| S3 — Rolling Window | ☐ | ☐ |

---

[[05-broadcasting]] | [[07-cheat-sheet]]
