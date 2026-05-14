# 📏 02 — Variance and Standard Deviation
## Measuring Spread

> [!info] Goal
> Learn how to measure how much values vary around the average.

---

## Why Spread Matters

Two datasets can have the same mean but very different behavior.

```python
import pandas as pd

a = pd.Series([48, 49, 50, 51, 52])
b = pd.Series([10, 30, 50, 70, 90])

print(a.mean(), b.mean())
print(a.std(), b.std())
```

Both have the same mean, but `b` is much more spread out.

---

## Range

The simplest spread measure:

```text
range = max - min
```

```python
scores = pd.Series([60, 70, 75, 80, 95])
print(scores.max() - scores.min())
```

Range is easy but sensitive to outliers.

---

## Variance

Variance measures average squared distance from the mean.

```python
print(scores.var())
```

Because variance is squared, its units are harder to interpret.

---

## Standard Deviation

Standard deviation is the square root of variance.

```python
print(scores.std())
```

It is easier to interpret because it uses the same unit as the original data.

---

## Interpretation

If average score is `80` and standard deviation is `5`, most scores are near 80.

If average score is `80` and standard deviation is `25`, scores vary widely.

---

## Sample vs Population

Pandas uses sample standard deviation by default:

```python
scores.std()       # ddof=1
scores.std(ddof=0) # population standard deviation
```

| Type | Use When |
|------|----------|
| Sample std | You have a sample from a larger population |
| Population std | You have the full population |

---

## Quantiles and IQR

The **interquartile range** is often used for skewed data.

```python
q1 = scores.quantile(0.25)
q3 = scores.quantile(0.75)
iqr = q3 - q1

print(iqr)
```

IQR measures the spread of the middle 50% of values.

---

## Data Science Uses

Spread helps with:

- detecting outliers
- comparing variability between groups
- feature scaling
- risk analysis
- understanding model inputs

---

## Common Mistakes

- Looking only at mean and ignoring spread.
- Comparing standard deviations across variables with different units.
- Treating high standard deviation as always bad.
- Forgetting outliers can inflate standard deviation.

---

## Practice

Given:

```python
daily_sales = pd.Series([100, 105, 98, 110, 500])
```

Tasks:

- calculate range
- calculate variance
- calculate standard deviation
- calculate IQR
- identify why the spread is large

---

## Interview Questions

**Q1:** What does standard deviation measure?

> It measures how spread out values are around the mean.

**Q2:** Why is standard deviation easier to interpret than variance?

> It has the same unit as the original data.

**Q3:** What is IQR?

> IQR is Q3 minus Q1, the spread of the middle 50% of data.

---

## ✅ Key Takeaways

- Mean tells center; standard deviation tells spread.
- Variance is squared spread.
- Standard deviation is more interpretable.
- IQR is useful for skewed data and outliers.

---

## 🔗 What's Next?

➡️ [[03-probability-basics]] — Understand uncertainty
