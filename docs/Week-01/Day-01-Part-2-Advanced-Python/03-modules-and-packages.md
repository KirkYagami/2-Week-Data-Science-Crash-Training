# 📦 03 – Modules and Packages

> **Prerequisites:** [[02-file-handling]]  
> **Time to read:** ~20 minutes

---

## 🧠 What is a Module?

A **module** is simply a `.py` file that contains Python code (functions, classes, variables). Modules let you split large programs into manageable, reusable pieces.

```
my_project/
├── main.py            ← your main script
├── utils.py           ← a module with helper functions
├── data_cleaner.py    ← a module with cleaning functions
└── models.py          ← a module with ML classes
```

---

## 📥 Importing Modules

```python
# Import entire module
import math
print(math.sqrt(16))       # 4.0
print(math.pi)             # 3.14159...

# Import specific names
from math import sqrt, pi
print(sqrt(16))            # 4.0 (no "math." prefix)
print(pi)                  # 3.14159...

# Import with alias (rename)
import numpy as np         # standard alias
import pandas as pd        # standard alias
import matplotlib.pyplot as plt  # standard alias

# Import everything (AVOID in production!)
from math import *
print(floor(3.7))          # 3 — works, but pollutes namespace
```

---

## 🗂️ Python Standard Library — Most Useful Modules

These come with Python — no installation needed!

### `os` — Operating System Interface

```python
import os

print(os.getcwd())                    # current directory
os.chdir("/path/to/dir")             # change directory
os.makedirs("output/2025", exist_ok=True)  # create nested dirs
print(os.listdir("."))               # list files in current dir
print(os.path.exists("file.txt"))    # check if file exists
print(os.path.join("data", "file.csv"))  # build path (os-safe)

# Environment variables (great for secrets)
api_key = os.environ.get("OPENAI_API_KEY", "not_set")
os.environ["MY_VAR"] = "hello"
```

### `sys` — System Parameters

```python
import sys

print(sys.version)          # Python version info
print(sys.argv)             # command-line arguments: ['script.py', 'arg1', 'arg2']
sys.exit(0)                 # exit program (0 = success)
print(sys.path)             # where Python looks for modules
```

### `datetime` — Date and Time

```python
from datetime import datetime, date, timedelta

# Current datetime
now = datetime.now()
today = date.today()
print(now)                         # 2025-01-15 14:30:22.123456
print(today)                       # 2025-01-15

# Create specific dates
birthday = date(1995, 6, 15)
meeting = datetime(2025, 3, 20, 14, 30)

# Format dates
print(now.strftime("%Y-%m-%d %H:%M"))        # 2025-01-15 14:30
print(now.strftime("%d/%m/%Y"))              # 15/01/2025
print(now.strftime("%B %d, %Y"))             # January 15, 2025

# Parse strings to dates
d = datetime.strptime("2025-01-15", "%Y-%m-%d")

# Date arithmetic
tomorrow = today + timedelta(days=1)
last_week = today - timedelta(weeks=1)
age_days = (today - birthday).days

# Time difference in data
from datetime import datetime
start = datetime(2025, 1, 1)
end = datetime(2025, 3, 20)
diff = end - start
print(f"Difference: {diff.days} days")
```

### `random` — Random Number Generation

```python
import random

random.seed(42)                          # reproducible randomness

print(random.random())                   # float in [0.0, 1.0)
print(random.randint(1, 10))             # int between 1 and 10 (inclusive)
print(random.choice(["a", "b", "c"]))   # random pick from list
print(random.choices(["a","b","c"], weights=[70, 20, 10], k=5))  # weighted choices

data = list(range(10))
random.shuffle(data)                     # shuffle IN PLACE
print(data)

sample = random.sample(range(100), k=5)  # pick k unique items
print(sample)
```

### `collections` — Specialized Containers

```python
from collections import Counter, defaultdict, OrderedDict, deque

# Counter — count frequencies
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
counts = Counter(words)
print(counts)                           # Counter({'apple': 3, 'banana': 2, 'cherry': 1})
print(counts.most_common(2))           # [('apple', 3), ('banana', 2)]

# defaultdict — dict with default value for missing keys
from collections import defaultdict
word_lengths = defaultdict(list)
for word in words:
    word_lengths[len(word)].append(word)
print(dict(word_lengths))
# {5: ['apple', 'apple', 'apple'], 6: ['banana', 'banana', 'cherry']}

# deque — fast double-ended queue
dq = deque([1, 2, 3])
dq.appendleft(0)                        # add to left: [0, 1, 2, 3]
dq.append(4)                            # add to right: [0, 1, 2, 3, 4]
dq.popleft()                            # remove from left: 0
```

### `itertools` — Iterator Tools

```python
import itertools

# chain — combine multiple iterables
combined = list(itertools.chain([1,2,3], [4,5,6], [7,8]))
print(combined)   # [1, 2, 3, 4, 5, 6, 7, 8]

# product — cartesian product
pairs = list(itertools.product(["A","B"], [1,2,3]))
print(pairs)   # [('A',1),('A',2),('A',3),('B',1),('B',2),('B',3)]

# combinations — all unique combos
combos = list(itertools.combinations([1,2,3,4], 2))
print(combos)   # [(1,2),(1,3),(1,4),(2,3),(2,3),(3,4)]

# groupby — group consecutive elements
from itertools import groupby
data = [("A", 1), ("A", 2), ("B", 3), ("B", 4), ("C", 5)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
```

---

## 📦 Creating Your Own Module

```python
# utils.py
"""Utility functions for the Data Science project."""

def normalize(values):
    """Normalize a list to [0, 1] range."""
    min_v, max_v = min(values), max(values)
    if min_v == max_v:
        return [0.5] * len(values)
    return [(v - min_v) / (max_v - min_v) for v in values]

def mean(values):
    """Calculate arithmetic mean."""
    return sum(values) / len(values) if values else None

CONSTANTS = {
    "version": "1.0",
    "author": "Your Name"
}
```

```python
# main.py
import utils

data = [10, 20, 30, 40, 50]
normalized = utils.normalize(data)
print(normalized)   # [0.0, 0.25, 0.5, 0.75, 1.0]

avg = utils.mean(data)
print(avg)          # 30.0
```

### `if __name__ == "__main__":`

This is a critical Python pattern — code inside this block only runs when the file is executed directly, not when imported:

```python
# utils.py
def normalize(values):
    min_v, max_v = min(values), max(values)
    return [(v - min_v) / (max_v - min_v) for v in values]

if __name__ == "__main__":
    # This runs ONLY when you run: python utils.py
    # It does NOT run when you do: import utils
    test_data = [10, 20, 30]
    print(normalize(test_data))   # Test the function
```

---

## 📦 Packages — Organizing Modules

A **package** is a directory of modules with an `__init__.py` file.

```
my_ds_project/
├── __init__.py
├── data/
│   ├── __init__.py
│   ├── loader.py
│   └── cleaner.py
├── models/
│   ├── __init__.py
│   ├── regression.py
│   └── classification.py
└── main.py
```

```python
# main.py
from data.loader import load_csv
from data.cleaner import clean_data
from models.regression import LinearModel

data = load_csv("sales.csv")
clean = clean_data(data)
model = LinearModel()
model.fit(clean)
```

---

## 🌐 Installing External Packages with pip

```bash
# Install a package
pip install pandas
pip install numpy scikit-learn matplotlib

# Install specific version
pip install pandas==2.0.0

# Install from requirements file
pip install -r requirements.txt

# List installed packages
pip list
pip show pandas

# Uninstall
pip uninstall pandas

# requirements.txt format
# numpy==1.24.0
# pandas>=2.0
# scikit-learn
# matplotlib
```

---

## ✅ Key Takeaways

- A **module** is any `.py` file; a **package** is a directory with `__init__.py`
- Use `import module` or `from module import name` to use other code
- Standard library (no install needed): `os`, `sys`, `datetime`, `random`, `collections`, `itertools`
- Always use `if __name__ == "__main__":` to protect runnable code
- Use `pip install` to add third-party packages
- Create a `requirements.txt` to share your project's dependencies

---

## 🔗 What's Next?

➡️ [[04-exception-handling]] — Handle errors gracefully so programs don't crash unexpectedly