# Modules and Packages

When a script grows past a few hundred lines, keeping everything in one file becomes a maintenance problem. Modules and packages are how Python solves this — they let you split code into named units that can be imported anywhere. Understanding this system also explains exactly what happens when you write `import pandas as pd` or `from sklearn.model_selection import train_test_split`.

**Prerequisites:** [[02-file-handling]]

## Learning Objectives

- Explain what a module is and how Python finds one when you import it
- Use `import`, `from x import y`, and aliases correctly
- Understand `__name__ == "__main__"` and why every script needs it
- Create your own module and import it from another file
- Know what a package is and how `__init__.py` controls imports
- Install third-party packages with `pip` and manage dependencies with `requirements.txt`
- Set up a virtual environment — and understand why you always should

---

## What is a Module?

A module is any `.py` file. When you write `import math`, Python finds a file called `math.py` (or a compiled equivalent) in its search path and executes it. Everything defined in that file becomes accessible as `math.something`.

That is the whole mechanism. There is no magic.

```python
# math is a module — a file with Python definitions
import math

print(math.sqrt(144))   # Output: 12.0
print(math.pi)          # Output: 3.141592653589793
print(math.floor(3.9))  # Output: 3
```

---

## Import Styles

```python
# Style 1: import the module, use dotted access
import json
data = json.loads('{"key": "value"}')

# Style 2: import specific names into local namespace
from pathlib import Path
from datetime import datetime, timedelta
report_path = Path("reports") / "january.csv"
today = datetime.now()

# Style 3: alias — standard for large libraries
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

arr = np.array([1, 2, 3])
df = pd.DataFrame({"x": arr})

# Style 4: import everything (avoid this)
from math import *  # pollutes your namespace with sqrt, sin, cos, pi, etc.
# If another import also defines 'pi', you now have a silent name collision
```

> [!warning]
> `from module import *` is tempting because it saves typing. In practice it makes code harder to read (where did `pi` come from?) and causes bugs when two modules export the same name. Use it only in interactive sessions, never in scripts or production code.

---

## Python's Most Useful Standard Library Modules

These ship with every Python installation — no `pip` needed.

### `os` — Operating System Interface

```python
import os

print(os.getcwd())                           # Output: /home/user/projects/ds-bootcamp

# Build paths (use pathlib instead for new code, but know this exists)
filepath = os.path.join("data", "raw", "customers.csv")
print(filepath)                              # Output: data/raw/customers.csv

# List directory contents
files = os.listdir(".")
print([f for f in files if f.endswith(".csv")])

# Create directories
os.makedirs("output/2025/q1", exist_ok=True)

# Environment variables — the correct way to access secrets and config
db_password = os.environ.get("DB_PASSWORD")  # None if not set
api_key = os.environ.get("OPENAI_API_KEY", "not-configured")

# Check if something exists
print(os.path.exists("data/customers.csv"))  # Output: True or False
print(os.path.isfile("data/customers.csv"))  # True for files
print(os.path.isdir("data"))                 # True for directories
```

### `sys` — System-Level Access

```python
import sys

print(sys.version)       # Output: 3.11.4 (main, ...
print(sys.platform)      # Output: linux, darwin, or win32

# Command-line arguments
# If you run: python train_model.py --epochs 100
print(sys.argv)          # Output: ['train_model.py', '--epochs', '100']

# Add a directory to Python's module search path
sys.path.append("/path/to/my/custom/modules")

# Exit the program — 0 means success, non-zero means failure
sys.exit(0)
```

### `datetime` — Date and Time Operations

```python
from datetime import datetime, date, timedelta

# Current moment
now = datetime.now()
today = date.today()

print(now)     # Output: 2025-01-15 14:30:22.123456
print(today)   # Output: 2025-01-15

# Create specific datetimes
campaign_start = datetime(2025, 1, 1, 9, 0)
campaign_end = datetime(2025, 3, 31, 23, 59)

# Format for display or file naming
timestamp_str = now.strftime("%Y%m%d_%H%M%S")
print(timestamp_str)                          # Output: 20250115_143022

readable = now.strftime("%B %d, %Y at %H:%M")
print(readable)                               # Output: January 15, 2025 at 14:30

# Parse a date string — common when reading from CSV
date_str = "2025-01-15"
parsed = datetime.strptime(date_str, "%Y-%m-%d")
print(parsed.year)   # Output: 2025

# Date arithmetic
next_week = today + timedelta(weeks=1)
thirty_days_ago = today - timedelta(days=30)

# Time difference
onboarding_date = date(2023, 6, 1)
tenure_days = (today - onboarding_date).days
print(f"Tenure: {tenure_days} days ({tenure_days // 365} years)")
```

### `collections` — Specialized Data Structures

```python
from collections import Counter, defaultdict, deque

# Counter — counts occurrences, most useful for frequency analysis
customer_tiers = ["gold", "silver", "gold", "bronze", "gold", "silver", "gold"]
tier_counts = Counter(customer_tiers)

print(tier_counts)                    # Output: Counter({'gold': 4, 'silver': 2, 'bronze': 1})
print(tier_counts["gold"])            # Output: 4
print(tier_counts.most_common(2))     # Output: [('gold', 4), ('silver', 2)]

# Combine two counters
additional = Counter(["bronze", "bronze", "silver"])
combined = tier_counts + additional
print(combined)  # Output: Counter({'gold': 4, 'silver': 3, 'bronze': 3})


# defaultdict — dict that creates a default value for missing keys
# Eliminates the "if key not in dict: dict[key] = []" boilerplate
department_employees = defaultdict(list)

employees = [
    ("Alice", "Engineering"), ("Bob", "Marketing"),
    ("Carol", "Engineering"), ("Dave", "Marketing"), ("Eve", "Engineering"),
]

for name, dept in employees:
    department_employees[dept].append(name)

print(dict(department_employees))
# Output: {'Engineering': ['Alice', 'Carol', 'Eve'], 'Marketing': ['Bob', 'Dave']}


# deque — fast appends and pops from both ends
# Use for queues, sliding windows, recent-N-items patterns
recent_errors = deque(maxlen=5)  # automatically drops oldest when full

for i in range(8):
    recent_errors.append(f"error_{i}")

print(list(recent_errors))   # Output: ['error_3', 'error_4', 'error_5', 'error_6', 'error_7']
```

### `itertools` — Efficient Iteration Combinators

```python
import itertools

# chain — flatten multiple iterables into one
week1_data = [10, 20, 30]
week2_data = [40, 50, 60]
week3_data = [70, 80, 90]
all_data = list(itertools.chain(week1_data, week2_data, week3_data))
print(all_data)  # Output: [10, 20, 30, 40, 50, 60, 70, 80, 90]


# product — cartesian product (all combinations)
# Useful for hyperparameter grid search
learning_rates = [0.001, 0.01, 0.1]
max_depths = [3, 5, 10]
param_grid = list(itertools.product(learning_rates, max_depths))
for lr, depth in param_grid:
    print(f"lr={lr}, max_depth={depth}")
# Output: lr=0.001, max_depth=3
#         lr=0.001, max_depth=5
#         ... 9 combinations total


# combinations — unique selections without replacement
feature_names = ["age", "income", "tenure"]
pairs = list(itertools.combinations(feature_names, 2))
print(pairs)
# Output: [('age', 'income'), ('age', 'tenure'), ('income', 'tenure')]


# islice — take only the first N items from any iterable (lazy)
import itertools
first_five = list(itertools.islice(range(10_000_000), 5))
print(first_five)  # Output: [0, 1, 2, 3, 4]
```

### `random` — Controlled Randomness

```python
import random

# Always set a seed for reproducible results in data science
random.seed(42)

print(random.random())                        # Output: 0.6394... (same every run with seed=42)
print(random.randint(1, 100))                 # Output: int in [1, 100] inclusive
print(random.choice(["train", "val", "test"])) # Output: random selection

feature_list = ["age", "income", "tenure", "plan_type", "city"]
random.shuffle(feature_list)                  # shuffles IN PLACE
print(feature_list)

# Sample k unique items
validation_indices = random.sample(range(1000), k=200)
print(len(validation_indices))  # Output: 200
```

---

## Creating Your Own Module

Any `.py` file is a module. Here is how to structure a real utility module:

```python
# data_utils.py
"""
Utility functions for data loading and basic statistics.
Import these functions wherever data cleaning is needed.
"""


def normalize(values: list) -> list:
    """Scale values to [0, 1] range using min-max normalization."""
    if len(values) == 0:
        return []
    min_v = min(values)
    max_v = max(values)
    if min_v == max_v:
        return [0.5] * len(values)
    return [(v - min_v) / (max_v - min_v) for v in values]


def mean(values: list) -> float | None:
    """Return arithmetic mean, or None if list is empty."""
    if not values:
        return None
    return sum(values) / len(values)


def remove_nulls(values: list) -> list:
    """Remove None values from a list."""
    return [v for v in values if v is not None]


VERSION = "1.0.0"
```

```python
# main.py — in the same directory as data_utils.py
import data_utils

raw_scores = [85, None, 92, 78, None, 88, 95]

clean_scores = data_utils.remove_nulls(raw_scores)
print(f"Clean scores: {clean_scores}")     # Output: [85, 92, 78, 88, 95]

avg = data_utils.mean(clean_scores)
print(f"Average: {avg}")                   # Output: 87.6

normalized = data_utils.normalize(clean_scores)
print(f"Normalized: {normalized}")         # Output: [0.41, 1.0, 0.0, 0.59, 1.0]... approx

# You can also import specific names
from data_utils import mean, normalize

print(mean([10, 20, 30]))  # Output: 20.0
```

---

## `if __name__ == "__main__":`

This is one of the most important patterns in Python. Every module has a `__name__` attribute. When Python runs a file directly, `__name__` is set to `"__main__"`. When it is imported, `__name__` is the module's filename.

```python
# data_utils.py — with proper guard

def normalize(values: list) -> list:
    if not values:
        return []
    min_v, max_v = min(values), max(values)
    if min_v == max_v:
        return [0.5] * len(values)
    return [(v - min_v) / (max_v - min_v) for v in values]


def mean(values: list) -> float | None:
    return sum(values) / len(values) if values else None


if __name__ == "__main__":
    # This block ONLY runs when you execute: python data_utils.py
    # It does NOT run when another file does: import data_utils
    test_scores = [40, 60, 80, 100]
    print("Testing normalize()...")
    print(normalize(test_scores))  # Output: [0.0, 0.333, 0.666, 1.0]
    print("Testing mean()...")
    print(mean(test_scores))       # Output: 70.0
    print("All tests passed.")
```

> [!tip]
> Put test code, demo runs, and argument parsing inside `if __name__ == "__main__":`. This lets other files import your functions without triggering the demo code. Every Python file you write that is meant to be both a module and a runnable script should have this guard.

---

## Packages — Organizing Multiple Modules

A **package** is a directory that contains multiple modules. The directory needs an `__init__.py` file to be recognized as a package (in modern Python, `__init__.py` can be empty, but it still needs to exist for some tools).

```
sales_analytics/
├── __init__.py
├── loaders/
│   ├── __init__.py
│   ├── csv_loader.py
│   └── json_loader.py
├── cleaning/
│   ├── __init__.py
│   ├── normalizer.py
│   └── deduplicator.py
└── reporting/
    ├── __init__.py
    └── summary.py
```

```python
# main.py — at the root of the project

from sales_analytics.loaders.csv_loader import load_customers
from sales_analytics.cleaning.normalizer import normalize_salaries
from sales_analytics.reporting.summary import generate_report

customers = load_customers("data/customers.csv")
customers = normalize_salaries(customers)
report = generate_report(customers)
```

You can also control what `from package import *` exposes by setting `__all__` in `__init__.py`:

```python
# sales_analytics/__init__.py
from .loaders.csv_loader import load_customers
from .cleaning.normalizer import normalize_salaries

__all__ = ["load_customers", "normalize_salaries"]
```

---

## Installing Packages with pip

```bash
# Install a package
pip install pandas

# Install a specific version
pip install pandas==2.0.0

# Install with a minimum version
pip install "scikit-learn>=1.3"

# Install multiple packages
pip install numpy pandas scikit-learn matplotlib seaborn

# Install from a requirements file
pip install -r requirements.txt

# List what is installed
pip list
pip show pandas         # details about one package

# Save current environment to a file
pip freeze > requirements.txt

# Uninstall
pip uninstall pandas
```

A `requirements.txt` looks like this:

```text
numpy==1.26.0
pandas==2.1.0
scikit-learn==1.3.2
matplotlib==3.8.0
seaborn==0.13.0
```

---

## Virtual Environments

A virtual environment is an isolated Python installation for one project. Without it, every project shares the same Python packages, and version conflicts are inevitable.

```bash
# Create a virtual environment
python -m venv venv

# Activate it
# Mac/Linux:
source venv/bin/activate

# Windows (PowerShell):
venv\Scripts\Activate.ps1

# Now pip install goes into the virtual environment, not the system Python
pip install pandas scikit-learn

# Save dependencies
pip freeze > requirements.txt

# Deactivate when done
deactivate
```

> [!warning]
> Never commit your `venv/` folder to git. Add it to `.gitignore`. It is hundreds of megabytes and completely machine-specific. Your `requirements.txt` is what gets committed — other developers recreate the environment from that file.

> [!tip]
> Get into the habit of creating a virtual environment for every project immediately after `git init`. The ten seconds it takes saves hours of debugging version conflicts later.

---

## Key Takeaways

> [!success]
> - A **module** is any `.py` file. A **package** is a directory of modules with `__init__.py`
> - Use `import module` to access via `module.name`; use `from module import name` for direct access
> - Avoid `from module import *` — it pollutes the namespace and hides where names came from
> - Always wrap runnable code in `if __name__ == "__main__":` so modules can be imported cleanly
> - The standard library (`os`, `sys`, `datetime`, `collections`, `itertools`) covers most common needs without any installation
> - Use `pip` to install third-party packages; use `requirements.txt` to share dependencies
> - Always use a virtual environment for each project

---

[[02-file-handling|← File Handling]] | [[04-exception-handling|Exception Handling →]]
