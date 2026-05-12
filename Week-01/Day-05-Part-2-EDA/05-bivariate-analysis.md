# 🔀 05 — Bivariate Analysis
## Studying Relationships Between Two Variables

> [!info] Goal
> Learn how to compare two variables and discover relationships.

---

## Numeric vs Numeric

Use scatter plots and correlation.

```python
sns.scatterplot(data=df, x="ad_spend", y="sales")
plt.show()

df[["ad_spend", "sales"]].corr()
```

Look for:

- positive relationship
- negative relationship
- no pattern
- clusters
- outliers

---

## Categorical vs Numeric

Use group summaries and box plots.

```python
df.groupby("category")["revenue"].mean().sort_values()
```

```python
sns.boxplot(data=df, x="category", y="revenue")
plt.xticks(rotation=45)
plt.show()
```

---

## Categorical vs Categorical

Use crosstabs.

```python
pd.crosstab(df["gender"], df["churn"])
```

Percentages:

```python
pd.crosstab(df["gender"], df["churn"], normalize="index") * 100
```

Chart:

```python
sns.countplot(data=df, x="gender", hue="churn")
plt.show()
```

---

## Time vs Numeric

```python
monthly = df.groupby("month")["revenue"].sum()
monthly.plot(kind="line", marker="o")
plt.show()
```

---

## Practice

Analyze:

- age vs spend
- city vs spend
- gender vs churn
- month vs revenue
- category vs revenue

Write one insight per relationship.

---

## Interview Questions

**Q1:** What is bivariate analysis?

> Analysis of the relationship between two variables.

**Q2:** Which chart compares numeric vs numeric?

> Scatter plot.

**Q3:** How do you compare categorical vs categorical?

> Crosstab or grouped count plot.

---

## 🔗 Next

➡️ [[06-eda-workflow]]
