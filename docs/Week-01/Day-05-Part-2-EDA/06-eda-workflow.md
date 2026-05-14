# 🧭 06 — EDA Workflow
## A Repeatable Exploration Process

> [!info] Goal
> Learn a structured EDA workflow you can reuse on any dataset.

---

## EDA Workflow

1. Understand the problem.
2. Load the data.
3. Inspect shape, columns, and types.
4. Clean obvious issues.
5. Analyze missing values.
6. Analyze individual features.
7. Analyze relationships.
8. Study target variable.
9. Summarize insights.
10. Decide next steps.

---

## Starter Template

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("data.csv")

print(df.head())
print(df.shape)
print(df.info())
print(df.isna().sum())
print(df.describe())
```

---

## Questions to Ask

Dataset:

- How many rows and columns?
- What does each row represent?
- What is the target?
- Are there duplicates?

Data quality:

- Missing values?
- Incorrect types?
- Impossible values?
- Outliers?

Analysis:

- Which features are important?
- Which variables relate to the target?
- Are there useful segments?
- What needs cleaning before modeling?

---

## Insight Format

Use this format:

```text
Observation: What the data shows.
Interpretation: Why it matters.
Action: What to do next.
```

Example:

```text
Observation: High-spend customers churn less often.
Interpretation: Engagement may be stronger among valuable customers.
Action: Investigate low-spend churn drivers separately.
```

---

## Deliverables

A good EDA notebook/report includes:

- dataset overview
- cleaning summary
- missing value summary
- univariate charts
- bivariate charts
- target analysis
- key insights
- recommended next steps

---

## Common Mistakes

- Plotting everything without questions.
- Skipping data quality checks.
- Ignoring target leakage.
- Reporting charts without insights.
- Doing EDA once and never revisiting it.

---

## Practice

Pick any dataset and produce:

- 5 data quality findings
- 5 univariate insights
- 5 bivariate insights
- 3 modeling recommendations

---

## Interview Questions

**Q1:** What is EDA?

> Exploratory Data Analysis: the process of understanding data before modeling or reporting.

**Q2:** What should EDA produce?

> Insights, data quality findings, feature understanding, and next steps.

**Q3:** Why is EDA important before modeling?

> It reveals data issues, patterns, leakage, outliers, and feature behavior.

---

## 🔗 Next

➡️ [[07-mini-case-study]]
