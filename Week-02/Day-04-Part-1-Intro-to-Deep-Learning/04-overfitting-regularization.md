# 🛡️ 04 — Overfitting and Regularization

Neural networks can memorize training data.

---

## Signs of Overfitting

- training loss decreases
- validation loss increases
- training accuracy much higher than validation accuracy

---

## Regularization Tools

### Dropout

```python
layers.Dropout(0.3)
```

### L2 Regularization

```python
layers.Dense(
    32,
    activation="relu",
    kernel_regularizer=keras.regularizers.l2(0.001)
)
```

### Early Stopping

```python
callback = keras.callbacks.EarlyStopping(
    patience=3,
    restore_best_weights=True
)
```

---

## Practical Advice

- start with a small model
- track validation loss
- use early stopping
- scale numeric inputs
- compare against simpler ML models

---

## Next

➡️ [[05-exercises]]
