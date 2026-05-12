# 🧭 02 — Supervised vs Unsupervised Learning

## Supervised Learning

Supervised learning uses labeled data: the target is known during training.

| Task | Target Type | Example |
|---|---|---|
| Regression | numeric | house price |
| Classification | category | churn yes/no |

```python
X = df[["age", "income", "tenure"]]
y = df["churn"]
```

---

## Unsupervised Learning

Unsupervised learning finds structure without a target.

Examples:

- customer segmentation
- anomaly detection
- topic discovery
- dimensionality reduction

```python
X = df[["spend", "visits", "tenure"]]
# no y
```

---

## Reinforcement Learning

An agent learns by taking actions and receiving rewards.

Examples:

- game-playing agents
- robotics
- recommendation policies

This crash course focuses mainly on supervised and unsupervised learning.

---

## Choosing the Learning Type

Ask:

```text
Do I have a target column?
```

If yes:

- numeric target -> regression
- categorical target -> classification

If no:

- clustering or dimensionality reduction

---

## Common Mistakes

- Calling every prediction problem "AI" without defining target.
- Using clustering when labeled data exists and prediction is the goal.
- Treating IDs as useful features.
- Forgetting that labels can be noisy.

---

## Practice

Classify each:

- predict sales next month
- predict whether a customer churns
- group users by behavior
- reduce 100 features to 2 for visualization

---

## Interview Questions

**Q1:** Difference between regression and classification?

> Regression predicts numeric values; classification predicts categories.

**Q2:** What is unsupervised learning?

> Learning patterns from data without target labels.

---

## Next

➡️ [[03-train-test-split-and-leakage]]
