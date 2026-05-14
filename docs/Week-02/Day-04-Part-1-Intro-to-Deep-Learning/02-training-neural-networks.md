# ⚙️ 02 — Training Neural Networks

Training means adjusting weights to reduce loss.

---

## Training Loop

1. Forward pass
2. Calculate loss
3. Backpropagation
4. Optimizer updates weights
5. Repeat for many epochs

---

## Key Terms

| Term | Meaning |
|---|---|
| Epoch | one pass through training data |
| Batch | subset of data used per update |
| Learning rate | update step size |
| Optimizer | algorithm that updates weights |
| Loss function | objective to minimize |

---

## Common Losses

| Task | Loss |
|---|---|
| Regression | MSE / MAE |
| Binary classification | binary cross-entropy |
| Multiclass classification | categorical cross-entropy |

---

## Common Problems

- learning rate too high -> unstable
- learning rate too low -> slow
- too many epochs -> overfitting
- too little data -> poor generalization

---

## Next

➡️ [[03-keras-quickstart]]
