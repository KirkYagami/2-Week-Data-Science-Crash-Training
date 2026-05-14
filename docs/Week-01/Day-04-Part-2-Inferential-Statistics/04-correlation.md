# 🔗 04 — Correlation
## Measuring Relationships Between Variables

> [!info] Goal
> Learn how to measure and interpret linear relationships between numeric variables.

---

## What is Correlation?

Correlation measures how two variables move together.

The most common correlation coefficient is **Pearson correlation**.

It ranges from `-1` to `1`.

| Value | Meaning |
|-------|---------|
| `1` | Perfect positive relationship |
| `0` | No linear relationship |
| `-1` | Perfect negative relationship |

---

## Example

```python
import pandas as pd

df = pd.DataFrame({
    "study_hours": [1, 2, 3, 4, 5],
    "score": [50, 60, 65, 75, 85],
    "sleep_hours": [8, 7, 7, 6, 5]
})

print(df.corr(numeric_only=True))
```

---

## Visualize First

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.scatterplot(data=df, x="study_hours", y="score")
plt.title("Study Hours vs Score")
plt.show()
```

Always inspect a scatter plot. Correlation alone can hide patterns.

---

## Correlation Heatmap

```python
sns.heatmap(df.corr(numeric_only=True), annot=True, cmap="coolwarm", center=0)
plt.title("Correlation Matrix")
plt.show()
```

---

## Correlation is Not Causation

If ice cream sales and drowning incidents are correlated, ice cream does not cause drowning.

Possible hidden factor:

```text
hot weather
```

Correlation suggests a relationship. It does not prove cause.

---

## Pearson vs Spearman

| Type | Use |
|------|-----|
| Pearson | Linear numeric relationship |
| Spearman | Monotonic/rank relationship |

```python
df.corr(method="pearson", numeric_only=True)
df.corr(method="spearman", numeric_only=True)
```

---

## Common Mistakes

- Assuming correlation proves causation.
- Ignoring non-linear relationships.
- Overreacting to weak correlations.
- Calculating correlation on categorical IDs.
- Not checking outliers.

---

## Practice

Create:

```python
data = pd.DataFrame({
    "ad_spend": [100, 200, 300, 400, 500],
    "sales": [1000, 1500, 2100, 2600, 3200],
    "discount": [5, 10, 10, 15, 20]
})
```

Tasks:

- calculate correlation matrix
- create a scatter plot of ad spend vs sales
- create a heatmap
- explain strongest relationship

---

## Interview Questions

**Q1:** What does correlation measure?

> The strength and direction of a relationship between variables.

**Q2:** Does correlation prove causation?

> No. Correlation does not prove cause and effect.

**Q3:** What does correlation near zero mean?

> There is little or no linear relationship.

---

## ✅ Key Takeaways

- Correlation ranges from -1 to 1.
- Positive correlation means variables move together.
- Negative correlation means one tends to rise as the other falls.
- Always visualize relationships.
- Correlation is not causation.

---

## 🔗 What's Next?

➡️ [[05-statistical-tests]] — Choose the right test
