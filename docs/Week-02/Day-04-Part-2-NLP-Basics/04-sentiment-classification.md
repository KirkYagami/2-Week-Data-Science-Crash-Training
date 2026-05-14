# 💬 04 — Sentiment Classification

## Example

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

df = pd.DataFrame({
    "text": [
        "I love this product",
        "This is terrible",
        "Amazing quality",
        "Not worth the money"
    ],
    "sentiment": [1, 0, 1, 0]
})

X_train, X_test, y_train, y_test = train_test_split(
    df["text"], df["sentiment"], test_size=0.25, random_state=42
)

pipe = Pipeline([
    ("tfidf", TfidfVectorizer()),
    ("model", LogisticRegression())
])

pipe.fit(X_train, y_train)
y_pred = pipe.predict(X_test)

print(classification_report(y_test, y_pred))
```

---

## Why Pipeline?

The vectorizer must learn vocabulary from training text only. Pipeline helps avoid leakage.

---

## Next

➡️ [[05-transformers-overview]]
