# 06 — NumPy Exercises
## Practice Problems with Solutions

> [!tip] How to Use This File
> 1. Read the problem carefully
> 2. **Try to solve it yourself first** — close the notes, open a Jupyter notebook
> 3. Only look at the solution when you're stuck or want to verify
> 4. Even if your solution works, read the provided solution — it may show a cleaner approach!

---

## 🧭 Table of Contents

- [[#Level 1 — Warm-Up (Array Basics)]]
- [[#Level 2 — Indexing & Slicing]]
- [[#Level 3 — Vectorization]]
- [[#Level 4 — Broadcasting]]
- [[#Level 5 — Mixed Challenge]]
- [[#Bonus Challenge — Real World]]

---

## Level 1 — Warm-Up (Array Basics)

### Exercise 1.1 — Create and Inspect

**Task:** Create a 1D array containing the even numbers from 2 to 20 (inclusive). Print its shape, size, dtype, and sum.

> [!example]- Solution 1.1
> ```python
> import numpy as np
> 
> arr = np.arange(2, 21, 2)
> # Or: np.linspace(2, 20, 10, dtype=int)
> 
> print(arr)          # [ 2  4  6  8 10 12 14 16 18 20]
> print(arr.shape)    # (10,)
> print(arr.size)     # 10
> print(arr.dtype)    # int64
> print(arr.sum())    # 110
> ```

---

### Exercise 1.2 — Build a Matrix

**Task:** Create a 5×5 matrix where:
- The border elements are 1
- The interior elements are 0

Expected output:
```
[[1 1 1 1 1]
 [1 0 0 0 1]
 [1 0 0 0 1]
 [1 0 0 0 1]
 [1 1 1 1 1]]
```

> [!example]- Solution 1.2
> ```python
> import numpy as np
> 
> mat = np.zeros((5, 5), dtype=int)
> mat[0, :]  = 1   # Top row
> mat[-1, :] = 1   # Bottom row
> mat[:, 0]  = 1   # Left column
> mat[:, -1] = 1   # Right column
> 
> print(mat)
> 
> # Alternative: start with ones, fill interior with zeros
> mat2 = np.ones((5, 5), dtype=int)
> mat2[1:-1, 1:-1] = 0
> print(mat2)
> ```

---

### Exercise 1.3 — Checkerboard

**Task:** Create an 8×8 matrix with a checkerboard pattern of 0s and 1s (like a chess board), where `(0,0)` is 0.

Expected (truncated):
```
[[0 1 0 1 0 1 0 1]
 [1 0 1 0 1 0 1 0]
 ...]
```

> [!example]- Solution 1.3
> ```python
> import numpy as np
> 
> # Method 1: using tile
> row_a = np.array([0, 1, 0, 1, 0, 1, 0, 1])
> row_b = np.array([1, 0, 1, 0, 1, 0, 1, 0])
> board = np.array([row_a, row_b] * 4)
> print(board)
> 
> # Method 2: elegant with modulo
> board2 = np.zeros((8, 8), dtype=int)
> board2[1::2, ::2]  = 1   # odd rows, even cols
> board2[::2,  1::2] = 1   # even rows, odd cols
> print(board2)
> 
> # Method 3: most concise
> board3 = np.indices((8, 8)).sum(axis=0) % 2
> print(board3)
> ```

---

### Exercise 1.4 — Array Reshape

**Task:** 
1. Create an array of 24 numbers from 1 to 24
2. Reshape it to (2, 3, 4)
3. Print the element at position [1, 2, 3]
4. Print the second "layer" (index 1) of the 3D array

> [!example]- Solution 1.4
> ```python
> import numpy as np
> 
> arr = np.arange(1, 25)           # [1, 2, ..., 24]
> tensor = arr.reshape(2, 3, 4)    # Shape: (2, 3, 4)
> 
> print(tensor)
> # [[[ 1  2  3  4]
> #   [ 5  6  7  8]
> #   [ 9 10 11 12]]
> #  [[13 14 15 16]
> #   [17 18 19 20]
> #   [21 22 23 24]]]
> 
> print(tensor[1, 2, 3])   # 24  ← layer 1, row 2, col 3
> print(tensor[1])          # Second layer
> # [[13 14 15 16]
> #  [17 18 19 20]
> #  [21 22 23 24]]
> ```

---

## Level 2 — Indexing & Slicing

### Exercise 2.1 — Slicing Patterns

**Task:** Given the array below, extract using slicing:
1. Every third element starting from index 0
2. The last 5 elements in reverse order
3. Elements at indices 2, 5, 8, 11 (every 3rd starting from 2)

```python
arr = np.arange(15)  # [0, 1, 2, ..., 14]
```

> [!example]- Solution 2.1
> ```python
> import numpy as np
> arr = np.arange(15)
> 
> # 1. Every third element from 0
> print(arr[::3])       # [ 0  3  6  9 12]
> 
> # 2. Last 5 elements in reverse
> print(arr[-1:-6:-1])  # [14 13 12 11 10]
> # or equivalently:
> print(arr[-5:][::-1]) # [14 13 12 11 10]
> 
> # 3. Indices 2, 5, 8, 11
> print(arr[2::3])      # [ 2  5  8 11]
> # or fancy indexing:
> print(arr[[2, 5, 8, 11]])  # [ 2  5  8 11]
> ```

---

### Exercise 2.2 — Matrix Surgery

**Task:** Given this 6×6 matrix, replace:
1. The entire main diagonal with -1
2. All elements in the bottom-right 3×3 submatrix with 99

```python
matrix = np.arange(36).reshape(6, 6)
```

> [!example]- Solution 2.2
> ```python
> import numpy as np
> matrix = np.arange(36).reshape(6, 6)
> print("Original:\n", matrix)
> 
> # 1. Replace main diagonal with -1
> np.fill_diagonal(matrix, -1)
> # Or: matrix[np.arange(6), np.arange(6)] = -1
> 
> # 2. Replace bottom-right 3×3 with 99
> matrix[3:, 3:] = 99
> 
> print("Modified:\n", matrix)
> # [[ -1   1   2   3   4   5]
> #  [  6  -1   8   9  10  11]
> #  [ 12  13  -1  15  16  17]
> #  [ 18  19  20  99  99  99]
> #  [ 24  25  26  99  99  99]
> #  [ 30  31  32  99  99  99]]
> ```

---

### Exercise 2.3 — Boolean Filtering

**Task:** Given an array of 20 random integers between -50 and 50:
1. Find all values between -10 and 10 (inclusive)
2. Replace all negative values with their absolute value
3. Count how many values are above the mean

> [!example]- Solution 2.3
> ```python
> import numpy as np
> np.random.seed(42)
> arr = np.random.randint(-50, 51, 20)
> print("Original:", arr)
> 
> # 1. Values between -10 and 10
> near_zero = arr[(arr >= -10) & (arr <= 10)]
> print("Near zero:", near_zero)
> 
> # 2. Replace negatives with absolute values
> arr_abs = arr.copy()               # Work on a copy!
> arr_abs[arr_abs < 0] = np.abs(arr_abs[arr_abs < 0])
> # Simpler:
> arr_abs = np.abs(arr)
> print("All positive:", arr_abs)
> 
> # 3. Count values above mean
> mean = arr.mean()
> count_above = np.sum(arr > mean)   # True = 1, False = 0
> print(f"Mean: {mean:.1f}, Values above mean: {count_above}")
> ```

---

## Level 3 — Vectorization

### Exercise 3.1 — Vectorize the Loop

**Task:** Rewrite this loop using NumPy vectorization. The function computes `result[i] = (data[i]^2 - mean) / std` for each element.

```python
import math

data = [5, 8, 2, 12, 3, 9, 7, 14, 1, 6]
mean = sum(data) / len(data)
variance = sum((x - mean)**2 for x in data) / len(data)
std = math.sqrt(variance)

result = []
for x in data:
    result.append((x**2 - mean) / std)
```

> [!example]- Solution 3.1
> ```python
> import numpy as np
> 
> data = np.array([5, 8, 2, 12, 3, 9, 7, 14, 1, 6])
> mean = data.mean()
> std  = data.std()
> 
> # One line!
> result = (data**2 - mean) / std
> 
> print(result.round(4))
> # Verify same as loop result
> ```

---

### Exercise 3.2 — Temperature Analysis

**Task:** You have temperature data for a week (7 days × 24 hours = 168 hourly readings).

```python
np.random.seed(0)
temperatures = 20 + np.random.randn(7, 24) * 5  # shape: (7, 24)
```

Answer these using vectorized NumPy (no loops!):
1. What is the average temperature for each day?
2. What is the hottest hour (column index) on average across all days?
3. How many individual hour readings were above 25°C?
4. What percentage of readings were between 18 and 28°C?
5. Find the day with the most temperature variation (highest std)

> [!example]- Solution 3.2
> ```python
> import numpy as np
> np.random.seed(0)
> temperatures = 20 + np.random.randn(7, 24) * 5
> 
> # 1. Average per day (average across hours = axis=1)
> daily_avg = temperatures.mean(axis=1)
> print("Daily averages:", daily_avg.round(2))
> 
> # 2. Hottest hour on average (average across days = axis=0)
> hourly_avg = temperatures.mean(axis=0)  # shape: (24,)
> hottest_hour = np.argmax(hourly_avg)
> print(f"Hottest hour: {hottest_hour} ({hourly_avg[hottest_hour]:.2f}°C)")
> 
> # 3. Readings above 25°C
> count_hot = np.sum(temperatures > 25)
> print(f"Hours above 25°C: {count_hot}")
> 
> # 4. Percentage between 18 and 28
> in_range = (temperatures >= 18) & (temperatures <= 28)
> percentage = in_range.mean() * 100  # mean of booleans = proportion
> print(f"In range [18,28]: {percentage:.1f}%")
> 
> # 5. Day with most variation
> daily_std = temperatures.std(axis=1)
> most_variable_day = np.argmax(daily_std)
> print(f"Most variable day: Day {most_variable_day} (std={daily_std[most_variable_day]:.2f})")
> ```

---

### Exercise 3.3 — Image Manipulation

**Task:** Simulate image operations on a grayscale image (256×256 array of pixel values 0-255):

```python
np.random.seed(42)
image = np.random.randint(0, 256, (256, 256), dtype=np.uint8)
```

1. Compute the histogram (count of each pixel value) without using `np.histogram`
2. Flip the image vertically
3. Increase brightness by 50 (clamped to 255 max)
4. Create a binary mask: 1 where pixel > 128, 0 elsewhere

> [!example]- Solution 3.3
> ```python
> import numpy as np
> np.random.seed(42)
> image = np.random.randint(0, 256, (256, 256), dtype=np.uint8)
> 
> # 1. Histogram (count occurrences of each value 0-255)
> hist = np.bincount(image.flatten(), minlength=256)
> print(f"Histogram shape: {hist.shape}, Total: {hist.sum()}")
> 
> # 2. Flip vertically (reverse rows)
> flipped = image[::-1, :]
> print(f"Original top-left: {image[0, 0]}, Flipped top-left: {flipped[0, 0]}")
> print(f"Original bottom-left: {image[-1, 0]}, Flipped bottom-left: {flipped[-1, 0]}")
> 
> # 3. Increase brightness by 50, clamped to 255
> brighter = np.clip(image.astype(np.int32) + 50, 0, 255).astype(np.uint8)
> # Must convert to int32 first to avoid overflow with uint8!
> 
> # 4. Binary mask
> mask = (image > 128).astype(np.uint8)  # or just (image > 128)
> print(f"Pixels > 128: {mask.sum()}, Fraction: {mask.mean():.3f}")
> ```

---

## Level 4 — Broadcasting

### Exercise 4.1 — Predict the Shape

**Without running code**, predict whether each operation is valid and what the output shape would be:

```python
A = np.ones((6, 1))
B = np.ones((1, 4))
C = np.ones((6, 4))
D = np.ones((4,))
E = np.ones((3, 6, 4))

# Predict: valid? output shape?
# 1. A + B
# 2. A + C
# 3. C + D
# 4. A + D
# 5. E + C
# 6. E + A
```

> [!example]- Solution 4.1
> ```
> 1. A(6,1) + B(1,4)  → (6,4)  ✅  Both dimensions compatible (6vs1, 1vs4)
> 2. A(6,1) + C(6,4)  → (6,4)  ✅  (6==6, 1→4)
> 3. C(6,4) + D(4,)   → (6,4)  ✅  D padded to (1,4) → (6,4)
> 4. A(6,1) + D(4,)   → (6,4)  ✅  D padded to (1,4); A(6,1)+D(1,4) → (6,4)
> 5. E(3,6,4) + C(6,4) → (3,6,4) ✅  C padded to (1,6,4) → (3,6,4)
> 6. E(3,6,4) + A(6,1) → (3,6,4) ✅  A padded to (1,6,1) → (3,6,4)
> ```

---

### Exercise 4.2 — Normalize Rows

**Task:** Given a matrix where each row represents a data point with multiple features, normalize each **row** so that the values in each row sum to 1 (row normalization, used in softmax, attention, etc.).

```python
matrix = np.array([[1, 2, 3],
                   [4, 0, 2],
                   [1, 1, 1]])
# Expected:
# row 0: [1/6, 2/6, 3/6] = [0.167, 0.333, 0.5]
# row 1: [4/6, 0/6, 2/6] = [0.667, 0.0,   0.333]
# row 2: [1/3, 1/3, 1/3] = [0.333, 0.333, 0.333]
```

> [!example]- Solution 4.2
> ```python
> import numpy as np
> 
> matrix = np.array([[1, 2, 3],
>                    [4, 0, 2],
>                    [1, 1, 1]], dtype=float)
> 
> # Row sums: shape (3,)
> row_sums = matrix.sum(axis=1)
> print("Row sums:", row_sums)  # [6. 6. 3.]
> 
> # Divide each row by its sum
> # row_sums shape: (3,) → needs to be (3, 1) for column broadcasting
> normalized = matrix / row_sums[:, np.newaxis]
> # Or: matrix / row_sums.reshape(-1, 1)
> 
> print(normalized.round(3))
> # [[0.167 0.333 0.5  ]
> #  [0.667 0.    0.333]
> #  [0.333 0.333 0.333]]
> 
> # Verify rows sum to 1
> print(normalized.sum(axis=1))  # [1. 1. 1.]
> ```

---

### Exercise 4.3 — Grade Curve

**Task:** A professor has grades for 4 students across 5 tests (shape: `(4, 5)`). 
1. Each test should be curved so the **maximum score becomes 100**. Apply this per test (column).
2. Then, each student's final grade is the weighted average of their 5 tests using weights `[0.1, 0.1, 0.2, 0.2, 0.4]`.
3. Assign letter grades: A (≥90), B (≥80), C (≥70), D (≥60), F (<60).

```python
grades = np.array([[70, 85, 60, 90, 75],
                   [80, 90, 70, 85, 95],
                   [60, 75, 80, 70, 65],
                   [90, 80, 90, 95, 88]])
```

> [!example]- Solution 4.3
> ```python
> import numpy as np
> 
> grades = np.array([[70, 85, 60, 90, 75],
>                    [80, 90, 70, 85, 95],
>                    [60, 75, 80, 70, 65],
>                    [90, 80, 90, 95, 88]], dtype=float)
> 
> # Step 1: Curve each test (column) so max becomes 100
> test_maxes = grades.max(axis=0)           # shape: (5,)
> curved = grades / test_maxes * 100        # Broadcasting: (4,5) / (5,) → (4,5)
> print("Curved grades:\n", curved.round(1))
> 
> # Step 2: Weighted average per student
> weights = np.array([0.1, 0.1, 0.2, 0.2, 0.4])
> # Weighted average = sum of (score * weight) for each student
> weighted_avg = (curved * weights).sum(axis=1)  # (4,5) * (5,) → (4,5) → sum → (4,)
> print("Final grades:", weighted_avg.round(2))
> 
> # Step 3: Letter grades using np.where chain
> letters = np.where(weighted_avg >= 90, 'A',
>           np.where(weighted_avg >= 80, 'B',
>           np.where(weighted_avg >= 70, 'C',
>           np.where(weighted_avg >= 60, 'D', 'F'))))
> print("Letter grades:", letters)
> ```

---

## Level 5 — Mixed Challenge

### Exercise 5.1 — Moving Average

**Task:** Compute the 3-day moving average of a stock price array (no loops!).

```python
prices = np.array([100, 102, 101, 104, 107, 105, 108, 110, 109, 112])
# Moving average window = 3
# Result should have length 8 (len - window + 1)
```

> [!example]- Solution 5.1
> ```python
> import numpy as np
> 
> prices = np.array([100, 102, 101, 104, 107, 105, 108, 110, 109, 112], dtype=float)
> window = 3
> 
> # Method 1: Using convolution
> kernel = np.ones(window) / window
> ma = np.convolve(prices, kernel, mode='valid')
> print("Moving average:", ma.round(2))
> # [101.   102.33 104.   105.33 106.67 107.67 109.   110.33]
> 
> # Method 2: Using strides (advanced)
> n = len(prices) - window + 1
> ma2 = np.array([prices[i:i+window].mean() for i in range(n)])
> # Wait, that's a loop... let's vectorize properly:
> 
> # Create index array and use fancy indexing
> indices = np.arange(window) + np.arange(n)[:, np.newaxis]  # (n, window)
> ma3 = prices[indices].mean(axis=1)
> print("Moving average (method 3):", ma3.round(2))
> ```

---

### Exercise 5.2 — Complete Data Pipeline

**Task:** Build a complete data pipeline:

```python
np.random.seed(123)
data = np.random.randn(100, 5)  # 100 samples, 5 features
```

Steps:
1. Find and print the indices of the 5 rows that have the largest L2 norm (Euclidean length)
2. Remove the top 5 outlier rows and create a clean dataset
3. Standardize (Z-score normalize) each feature in the clean dataset
4. Compute the correlation between features 0 and 1 using only vectorized NumPy

> [!example]- Solution 5.2
> ```python
> import numpy as np
> np.random.seed(123)
> data = np.random.randn(100, 5)
> 
> # Step 1: L2 norm of each row
> norms = np.sqrt((data**2).sum(axis=1))    # or np.linalg.norm(data, axis=1)
> top5_indices = np.argsort(norms)[-5:]     # indices of 5 largest norms
> print("Top 5 outlier indices:", np.sort(top5_indices))
> 
> # Step 2: Remove outliers
> all_indices = np.arange(100)
> clean_indices = np.setdiff1d(all_indices, top5_indices)
> clean_data = data[clean_indices]          # Fancy indexing → copy
> print(f"Clean data shape: {clean_data.shape}")  # (95, 5)
> 
> # Step 3: Z-score normalization per feature (column)
> col_mean = clean_data.mean(axis=0)        # shape: (5,)
> col_std  = clean_data.std(axis=0)         # shape: (5,)
> standardized = (clean_data - col_mean) / col_std  # broadcasting
> 
> print("After standardization:")
> print(f"  Means: {standardized.mean(axis=0).round(6)}")   # All ~0
> print(f"  Stds:  {standardized.std(axis=0).round(6)}")    # All ~1
> 
> # Step 4: Correlation between feature 0 and feature 1
> f0 = standardized[:, 0]
> f1 = standardized[:, 1]
> correlation = np.dot(f0, f1) / (len(f0) - 1)  # Pearson r (z-score data)
> # Or using np.corrcoef:
> corr_matrix = np.corrcoef(f0, f1)
> print(f"Correlation (manual): {correlation:.4f}")
> print(f"Correlation (np.corrcoef): {corr_matrix[0, 1]:.4f}")
> ```

---

## Bonus Challenge — Real World

### 🎯 Challenge: Image Histogram Equalization

**Background:** Histogram equalization is a technique to improve the contrast of an image. The idea: redistribute pixel intensities so they're more uniformly spread.

**Task:**

```python
np.random.seed(0)
# Simulate a "dark" image — most pixels clustered in low values
dark_image = (np.random.exponential(scale=50, size=(64, 64))).clip(0, 255).astype(np.uint8)
```

1. Plot the histogram (distribution of pixel values) — just print bucket counts
2. Apply histogram equalization using only NumPy (no OpenCV/PIL)
3. Verify that the equalized image has a more uniform distribution

**Hint:** Equalization formula: `equalized[i] = round((CDF[pixel[i]] - CDF_min) / (N - CDF_min) * 255)`
where CDF is the cumulative distribution function.

> [!example]- Solution — Bonus
> ```python
> import numpy as np
> 
> np.random.seed(0)
> dark_image = (np.random.exponential(scale=50, size=(64, 64))).clip(0, 255).astype(np.uint8)
> 
> print("=== Original Image ===")
> print(f"Shape: {dark_image.shape}")
> print(f"Min: {dark_image.min()}, Max: {dark_image.max()}, Mean: {dark_image.mean():.1f}")
> 
> # Step 1: Histogram (count of each pixel value 0-255)
> hist = np.bincount(dark_image.flatten(), minlength=256)
> print(f"\nHistogram (first 10 buckets): {hist[:10]}")
> print(f"Total pixels: {hist.sum()}")
> 
> # Step 2: Histogram equalization
> N = dark_image.size   # total number of pixels
> 
> # Cumulative Distribution Function (CDF)
> cdf = hist.cumsum()   # cumsum of histogram
> 
> # CDF_min: smallest non-zero CDF value
> cdf_min = cdf[cdf > 0].min()
> 
> # Apply equalization formula using NumPy vectorization
> # Create a lookup table: for each pixel value 0-255, compute equalized value
> lookup = np.round((cdf - cdf_min) / (N - cdf_min) * 255).astype(np.uint8)
> 
> # Apply lookup table to image (fancy indexing!)
> equalized = lookup[dark_image]   # dark_image values used as indices into lookup!
> 
> print("\n=== Equalized Image ===")
> print(f"Min: {equalized.min()}, Max: {equalized.max()}, Mean: {equalized.mean():.1f}")
> 
> # Step 3: Compare histograms
> hist_eq = np.bincount(equalized.flatten(), minlength=256)
> print(f"\nOriginal - pixels below 64: {hist[:64].sum()} / {N}")
> print(f"Equalized - pixels below 64: {hist_eq[:64].sum()} / {N}")
> print(f"Equalized - pixels above 192: {hist_eq[192:].sum()} / {N}")
> # The equalized image should have more even distribution!
> ```

---

## 📊 Progress Tracker

| Exercise | Solved Without Help? | Understood Solution? |
|----------|---------------------|---------------------|
| 1.1 Array Basics | ☐ | ☐ |
| 1.2 Border Matrix | ☐ | ☐ |
| 1.3 Checkerboard | ☐ | ☐ |
| 1.4 Reshape | ☐ | ☐ |
| 2.1 Slicing | ☐ | ☐ |
| 2.2 Matrix Surgery | ☐ | ☐ |
| 2.3 Boolean Filter | ☐ | ☐ |
| 3.1 Vectorize Loop | ☐ | ☐ |
| 3.2 Temperature | ☐ | ☐ |
| 3.3 Image Ops | ☐ | ☐ |
| 4.1 Predict Shape | ☐ | ☐ |
| 4.2 Normalize Rows | ☐ | ☐ |
| 4.3 Grade Curve | ☐ | ☐ |
| 5.1 Moving Average | ☐ | ☐ |
| 5.2 Data Pipeline | ☐ | ☐ |
| Bonus | ☐ | ☐ |

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[05-broadcasting]] | [[07-cheat-sheet]] |

---

*Tags: #numpy #exercises #practice #vectorization #broadcasting #indexing*