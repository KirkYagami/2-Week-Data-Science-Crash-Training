# 🤖 01 — What is Machine Learning?

Machine learning is a way to teach computers patterns from data instead of manually writing every rule.

Traditional programming:

```text
rules + data -> output
```

Machine learning:

```text
data + output examples -> learned rules/model
```

---

## Examples

| Problem | Input Features | Target |
|---|---|---|
| Predict house price | area, rooms, location | price |
| Predict churn | tenure, plan, usage | churn yes/no |
| Classify email | words, sender, links | spam/not spam |
| Recommend movies | ratings, genres, users | suggested movie |

---

## Core Vocabulary

| Term | Meaning |
|---|---|
| Dataset | Collection of examples |
| Row / sample | One observation |
| Feature | Input column |
| Target / label | Value to predict |
| Model | Learned pattern |
| Training | Fitting model on data |
| Prediction | Model output for new data |
| Evaluation | Measuring model quality |

---

## The ML Project Loop

1. Define the problem.
2. Collect data.
3. Explore and clean data.
4. Split into train/test sets.
5. Train model.
6. Evaluate model.
7. Improve features/model.
8. Communicate results.

---

## ML is Not Magic

ML models learn from patterns present in training data. If the data is biased, incomplete, leaked, or noisy, the model will inherit those problems.

Good data science is often more about data quality and evaluation than model choice.

---

## Practice

For each problem, identify features and target:

- predicting exam score
- predicting loan default
- grouping customers into segments
- detecting fake reviews

---

## Interview Questions

**Q1:** What is machine learning?

> A method where computers learn patterns from data to make predictions or decisions.

**Q2:** What is a feature?

> An input variable used by the model.

**Q3:** What is a target?

> The value the model is trained to predict.

---

## Next

➡️ [[02-supervised-unsupervised]]
