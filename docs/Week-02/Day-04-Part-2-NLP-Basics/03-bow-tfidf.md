# 🔤 03 — Bag of Words and TF-IDF

## Bag of Words

Counts word occurrences.

```python
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer()
X = vectorizer.fit_transform(texts)
```

---

## TF-IDF

TF-IDF gives higher weight to words that are important in a document but not common everywhere.

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer(max_features=5000, stop_words="english")
X = vectorizer.fit_transform(texts)
```

---

## N-Grams

```python
TfidfVectorizer(ngram_range=(1, 2))
```

This captures single words and two-word phrases.

---

## Next

➡️ [[04-sentiment-classification]]
