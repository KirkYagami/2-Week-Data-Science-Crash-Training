# 🧪 06 — Practice Questions: Statistics Basics

---

## Setup

```python
import pandas as pd
import numpy as np
```

---

## Exercise 1 — Central Tendency

```python
sales = pd.Series([100, 120, 130, 125, 1000])
```

Tasks:

- calculate mean
- calculate median
- explain which better represents typical sales

### Solution

```python
print(sales.mean())
print(sales.median())
```

Median is better because `1000` is an outlier.

---

## Exercise 2 — Spread

```python
scores = pd.Series([60, 65, 70, 75, 80, 85, 90])
```

Tasks:

- calculate range
- calculate variance
- calculate standard deviation
- calculate IQR

### Solution

```python
print(scores.max() - scores.min())
print(scores.var())
print(scores.std())
print(scores.quantile(0.75) - scores.quantile(0.25))
```

---

## Exercise 3 — Probability

Questions:

- probability of heads in a fair coin flip
- probability of rolling a 6 on a fair die
- probability of two heads in two fair coin flips

### Solution

```text
P(heads) = 1/2
P(rolling 6) = 1/6
P(two heads) = 1/2 * 1/2 = 1/4
```

---

## Exercise 4 — Simulate Coin Flips

```python
flips = np.random.choice(["H", "T"], size=10000)
prob_heads = (flips == "H").mean()

print(prob_heads)
```

The result should be close to `0.5`.

---

## Exercise 5 — Distribution Shape

```python
normal = pd.Series(np.random.normal(50, 10, 1000))
skewed = pd.Series(np.random.exponential(10, 1000))
```

Tasks:

- plot histograms
- compare mean and median
- calculate skew

### Solution

```python
normal.hist(bins=30)
skewed.hist(bins=30)

print(normal.mean(), normal.median(), normal.skew())
print(skewed.mean(), skewed.median(), skewed.skew())
```

---

## Final Challenge

Given:

```python
customers = pd.DataFrame({
    "age": [21, 25, 29, 35, 40, 42, 80],
    "spend": [500, 700, 650, 800, 900, 1200, 10000],
    "segment": ["A", "B", "A", "A", "B", "B", "A"]
})
```

Tasks:

- summarize `age` and `spend`
- identify which column has stronger outliers
- calculate mode of segment
- compare mean vs median spend
- write 2 insights

### One Possible Solution

```python
print(customers.describe())
print(customers["segment"].mode())
print(customers["spend"].mean())
print(customers["spend"].median())
```

Insights:

- Spend is strongly right-skewed because of `10000`.
- Median spend is more representative than mean spend.

---

## ✅ Self-Check

- [ ] I can calculate mean, median, and mode
- [ ] I can calculate variance and standard deviation
- [ ] I can explain IQR
- [ ] I can calculate simple probabilities
- [ ] I can identify skew from a histogram
- [ ] I can choose mean or median appropriately

---

## 🔗 Next

➡️ [[../Day-04-Part-2-Inferential-Statistics/01-hypothesis-testing|Inferential Statistics]]
