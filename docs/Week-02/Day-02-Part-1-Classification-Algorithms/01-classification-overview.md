# 🧭 01 — Classification Overview

Classification predicts a category.

Examples:

- spam vs not spam
- churn vs not churn
- fraud vs legitimate
- disease vs no disease
- sentiment: positive, neutral, negative

---

## Binary vs Multiclass

| Type | Example |
|---|---|
| Binary | churn yes/no |
| Multiclass | low/medium/high risk |
| Multilabel | movie can be comedy and drama |

---

## Classification Workflow

```python
X = df.drop(columns=["churn"])
y = df["churn"]
```

1. split with stratification
2. preprocess features
3. train classifier
4. predict labels/probabilities
5. evaluate with suitable metrics
6. inspect confusion matrix and errors

---

## Common Models

| Model | Good For |
|---|---|
| Logistic Regression | interpretable baseline |
| KNN | simple distance-based classification |
| Naive Bayes | text and simple probabilistic tasks |
| Decision Tree | interpretable nonlinear rules |
| Random Forest | strong general-purpose baseline |
| Gradient Boosting | high-performing tabular model |

---

## Next

➡️ [[02-logistic-regression]]
