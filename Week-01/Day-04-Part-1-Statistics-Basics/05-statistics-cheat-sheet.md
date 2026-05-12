# ⚡ 05 — Statistics Cheat Sheet
## Quick Reference

---

## Central Tendency

| Measure | Formula / Pandas | Use |
|---------|------------------|-----|
| Mean | `s.mean()` | Average value |
| Median | `s.median()` | Typical value with outliers |
| Mode | `s.mode()` | Most common value |

---

## Spread

| Measure | Pandas | Meaning |
|---------|--------|---------|
| Range | `s.max() - s.min()` | Full spread |
| Variance | `s.var()` | Average squared spread |
| Standard deviation | `s.std()` | Typical distance from mean |
| IQR | `s.quantile(.75) - s.quantile(.25)` | Middle 50% spread |

---

## Summary Commands

```python
s.describe()
df.describe()
df.describe(include="all")
df["col"].value_counts()
df["col"].value_counts(normalize=True)
```

---

## Probability

```text
P(not A) = 1 - P(A)
P(A and B) = P(A) * P(B)    # if independent
P(A or B) = P(A) + P(B) - P(A and B)
```

---

## Common Distributions

| Distribution | Use |
|--------------|-----|
| Normal | Symmetric bell-shaped values |
| Uniform | Equal likelihood |
| Binomial | Count of successes |
| Exponential | Waiting time / right-skewed data |

---

## Shape Checks

```python
df["col"].hist(bins=30)
df["col"].skew()
df["col"].plot(kind="box")
```

| Skew | Meaning |
|------|---------|
| `> 0` | Right-skewed |
| `< 0` | Left-skewed |
| near `0` | Roughly symmetric |

---

## Choosing Mean vs Median

| Situation | Prefer |
|-----------|--------|
| Symmetric data | Mean |
| Skewed data | Median |
| Outliers present | Median |
| Categorical data | Mode |

---

## Outlier Rule

```python
q1 = s.quantile(0.25)
q3 = s.quantile(0.75)
iqr = q3 - q1

lower = q1 - 1.5 * iqr
upper = q3 + 1.5 * iqr

outliers = s[(s < lower) | (s > upper)]
```

---

## Interview Quick Answers

**Mean vs median?**

> Mean is average; median is the middle value and is more robust to outliers.

**Variance vs standard deviation?**

> Variance is squared spread; standard deviation is its square root and easier to interpret.

**Why visualize distributions?**

> To detect skew, outliers, and whether summary statistics are appropriate.

---

## 🔗 What's Next?

➡️ [[06-practice-questions]] — Practice the concepts
