# 📊 01 — Mean, Median, and Mode
## Measures of Central Tendency

> [!info] Goal
> Learn how to summarize the "typical" value in a dataset and choose the right measure for the situation.

---

## Why Central Tendency Matters

Central tendency helps answer questions like:

- What is the average salary?
- What is the typical house price?
- What score did most students get?
- What is the usual customer spend?

The three most common measures are:

| Measure | Meaning | Best When |
|---------|---------|-----------|
| Mean | Arithmetic average | Data has no extreme outliers |
| Median | Middle value | Data is skewed or has outliers |
| Mode | Most frequent value | Categorical or repeated values |

---

## Mean

The **mean** is the sum of all values divided by the number of values.

```text
mean = sum(values) / count(values)
```

```python
import pandas as pd

salaries = pd.Series([30000, 40000, 50000, 60000, 1000000])

print(salaries.mean())
```

The mean is sensitive to outliers. Here, `1000000` pulls the average upward.

---

## Median

The **median** is the middle value after sorting.

```python
print(salaries.median())
```

For skewed data like income, house prices, or spending, median is often more representative than mean.

---

## Mode

The **mode** is the most common value.

```python
cities = pd.Series(["Delhi", "Mumbai", "Delhi", "Pune", "Delhi"])

print(cities.mode())
```

Mode works well for categorical data.

---

## Mean vs Median Example

```python
prices = pd.Series([20, 25, 30, 35, 1000])

print("Mean:", prices.mean())
print("Median:", prices.median())
```

The mean is affected by `1000`; the median remains closer to the typical value.

---

## In Pandas DataFrames

```python
df = pd.DataFrame({
    "age": [22, 25, 29, 31, 35],
    "salary": [30000, 40000, 50000, 60000, 1000000],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Delhi"]
})

print(df["age"].mean())
print(df["salary"].median())
print(df["city"].mode()[0])
```

---

## Common Mistakes

- Using mean for heavily skewed data.
- Forgetting that missing values are ignored by Pandas by default.
- Using mode without checking if there are multiple modes.
- Reporting an average without units or context.

---

## Practice

Given:

```python
customer_spend = pd.Series([500, 700, 650, 800, 10000])
```

Tasks:

- calculate mean
- calculate median
- explain which is better here
- add one more extreme value and observe the change

---

## Interview Questions

**Q1:** When is median better than mean?

> Median is better when data is skewed or contains outliers.

**Q2:** Can a dataset have more than one mode?

> Yes. Multiple values can share the highest frequency.

**Q3:** Why can mean be misleading?

> Extreme values can pull the mean away from the typical value.

---

## ✅ Key Takeaways

- Mean is useful but sensitive to outliers.
- Median is robust for skewed data.
- Mode is useful for repeated or categorical values.
- Always choose the summary statistic based on the data shape.

---

## 🔗 What's Next?

➡️ [[02-variance-and-std]] — Measure how spread out the data is
