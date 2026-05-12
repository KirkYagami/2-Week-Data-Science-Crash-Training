# 🧪 05 — Exercises: Machine Learning Basics

## Exercise 1 — Identify Problem Type

Classify each as regression, classification, clustering, or dimensionality reduction:

- predict house price
- detect spam email
- group customers by behavior
- compress 50 features into 2

## Exercise 2 — Train/Test Split

```python
from sklearn.datasets import load_diabetes
from sklearn.model_selection import train_test_split

data = load_diabetes(as_frame=True)
X = data.data
y = data.target
```

Split into 80% train and 20% test.

## Exercise 3 — First Model

Train a decision tree on the Iris dataset and report accuracy.

## Exercise 4 — Leakage Check

Explain why scaling before train/test split can be leakage.

---

## Self-Check

- [ ] I can define features and target
- [ ] I can choose supervised vs unsupervised
- [ ] I can split train/test safely
- [ ] I can train a first scikit-learn model

---

## Next

➡️ [[../Day-01-Part-2-Regression-Algorithms/00-agenda|Regression Algorithms]]
