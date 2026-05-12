# 🔍 04 — Univariate Analysis
## Studying One Variable at a Time

> [!info] Goal
> Learn how to analyze individual numeric and categorical columns.

---

## Numeric Features

Use summary statistics:

```python
df["revenue"].describe()
df["revenue"].skew()
```

Use charts:

```python
sns.histplot(df["revenue"], bins=30, kde=True)
plt.show()

sns.boxplot(x=df["revenue"])
plt.show()
```

Look for:

- center
- spread
- skew
- outliers
- impossible values

---

## Categorical Features

```python
df["city"].value_counts()
df["city"].value_counts(normalize=True) * 100
```

Chart:

```python
top_cities = df["city"].value_counts().head(10)
sns.barplot(x=top_cities.values, y=top_cities.index)
plt.show()
```

Look for:

- dominant categories
- rare categories
- inconsistent labels
- missing values

---

## Date Features

```python
df["order_date"] = pd.to_datetime(df["order_date"])
df["month"] = df["order_date"].dt.month
df["day_name"] = df["order_date"].dt.day_name()
```

Analyze trends:

```python
df["month"].value_counts().sort_index()
```

---

## Univariate Checklist

- [ ] data type
- [ ] missing values
- [ ] unique count
- [ ] summary statistics
- [ ] distribution
- [ ] outliers
- [ ] common categories
- [ ] rare categories

---

## Practice

For a customer dataset:

- analyze age
- analyze spend
- analyze city
- analyze signup month
- write one insight per feature

---

## Interview Questions

**Q1:** What is univariate analysis?

> Analysis of one variable at a time.

**Q2:** What chart is useful for numeric distribution?

> Histogram or box plot.

**Q3:** What function counts categorical values?

> `value_counts()`.

---

## 🔗 Next

➡️ [[05-bivariate-analysis]]
