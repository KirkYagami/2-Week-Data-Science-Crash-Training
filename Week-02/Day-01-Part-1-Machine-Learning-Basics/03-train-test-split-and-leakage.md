# ✂️ 03 — Train/Test Split and Leakage

> A model must be evaluated on data it did not learn from.

---

## Why Split Data?

If you evaluate on the same data used for training, the score can look artificially good.

```text
train data -> learn model
test data  -> estimate real-world performance
```

---

## Basic Split

```python
from sklearn.model_selection import train_test_split

X = df.drop(columns=["target"])
y = df["target"]

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42
)
```

---

## Stratified Split

For classification, preserve class balance:

```python
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y
)
```

---

## Data Leakage

Leakage happens when training data contains information that would not be available at prediction time.

Examples:

- `cancellation_date` used to predict churn
- scaling before train/test split
- filling missing values using full dataset statistics
- target-derived columns accidentally included as features

---

## Safe Preprocessing Rule

Fit preprocessing only on training data.

```text
fit on train
transform train
transform test
```

Scikit-learn pipelines help enforce this.

---

## Time-Based Split

For time series, do not shuffle future into past.

```python
train = df[df["date"] < "2025-01-01"]
test = df[df["date"] >= "2025-01-01"]
```

---

## Practice

Identify leakage:

- using future sales to predict current sales
- normalizing before splitting
- including customer_id as numeric feature
- filling missing age using median from training set only

---

## Interview Questions

**Q1:** Why split train and test data?

> To estimate how the model performs on unseen data.

**Q2:** What is data leakage?

> When information unavailable in real prediction leaks into training or evaluation.

---

## Next

➡️ [[04-scikit-learn-workflow]]
