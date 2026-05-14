# 🔍 02 — EDA and Cleaning

## First Checks

```python
df.head()
df.shape
df.info()
df.isna().sum()
df["churn"].value_counts(normalize=True)
```

## Cleaning Tasks

- standardize column names
- fix data types
- handle missing values
- remove exact duplicates
- inspect invalid values
- document decisions

## EDA Questions

- What is the churn rate?
- Which customer segments churn most?
- How does tenure relate to churn?
- How does monthly spend relate to churn?
- Are there missing values related to churn?

## Charts

- churn count plot
- numeric distributions
- box plots by churn
- correlation heatmap

---

## Next

➡️ [[03-feature-engineering]]
