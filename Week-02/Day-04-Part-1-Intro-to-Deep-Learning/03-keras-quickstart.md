# 🚀 03 — Keras Quickstart

Keras is a beginner-friendly deep learning API.

---

## Binary Classification Example

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

model = keras.Sequential([
    layers.Dense(32, activation="relu", input_shape=(X_train.shape[1],)),
    layers.Dense(16, activation="relu"),
    layers.Dense(1, activation="sigmoid")
])

model.compile(
    optimizer="adam",
    loss="binary_crossentropy",
    metrics=["accuracy"]
)

history = model.fit(
    X_train,
    y_train,
    validation_split=0.2,
    epochs=20,
    batch_size=32
)
```

---

## Evaluate

```python
test_loss, test_acc = model.evaluate(X_test, y_test)
print(test_acc)
```

---

## Predict

```python
proba = model.predict(X_test)
pred = (proba >= 0.5).astype(int)
```

---

## Next

➡️ [[04-overfitting-regularization]]
