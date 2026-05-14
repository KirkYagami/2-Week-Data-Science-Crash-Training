# 🧹 02 — Text Preprocessing

Common preprocessing steps:

- lowercase
- remove extra whitespace
- remove punctuation if useful
- remove stop words
- tokenize
- lemmatize/stem

---

## Simple Cleaning

```python
text = "  This movie was AMAZING!!!  "
clean = text.strip().lower()
```

With Pandas:

```python
df["text_clean"] = df["text"].str.strip().str.lower()
```

---

## What Not to Remove Blindly

Do not always remove:

- negation words like "not"
- punctuation in sentiment tasks
- casing if it carries meaning
- emojis if they matter

Preprocessing depends on the task.

---

## Next

➡️ [[03-bow-tfidf]]
