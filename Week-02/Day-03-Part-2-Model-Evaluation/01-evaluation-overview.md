# 🧭 01 — Evaluation Overview

Model evaluation answers:

```text
How well will this model perform on new data?
```

---

## Good Evaluation Requires

- unseen test data
- appropriate metric
- no leakage
- comparison to baseline
- error analysis
- business interpretation

---

## Baseline Model

Regression baseline:

```python
from sklearn.dummy import DummyRegressor
```

Classification baseline:

```python
from sklearn.dummy import DummyClassifier
```

If your model cannot beat a dummy baseline, it is not useful.

---

## Next

➡️ [[02-cross-validation]]
