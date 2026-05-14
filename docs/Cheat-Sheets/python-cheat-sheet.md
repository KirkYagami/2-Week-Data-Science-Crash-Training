# Python Cheat Sheet

A practitioner's reference for data science Python. Each entry explains when to use the pattern, then shows a runnable snippet. Scan it, don't memorize it — return here when you're in the middle of real work.

---

## 1. Data Types & Variables

### Numeric types: int and float

Use `int` for counts, indices, and discrete values. Use `float` when you need decimals or when a calculation might produce one. Python 3 division always returns a float — use `//` when you want integer division.

```python
rows = 1000          # int — row count
learning_rate = 0.01 # float — model hyperparameter

print(type(rows))           # <class 'int'>
print(type(learning_rate))  # <class 'float'>

print(7 / 2)   # 3.5   — true division
print(7 // 2)  # 3     — floor division
print(7 % 2)   # 1     — remainder
```

### Strings and booleans

`str` is immutable. `bool` is a subclass of `int` — `True == 1` and `False == 0`, which matters when summing boolean masks in NumPy/pandas.

```python
label = "churn"
is_valid = True

print(type(label))     # <class 'str'>
print(int(is_valid))   # 1
print(True + True)     # 2  — useful for counting matches in a list
```

### None — the absence of a value

`None` is not zero, not an empty string, not `False`. It signals that a value is missing or not yet assigned. Use `is None` for checks, not `== None`.

```python
result = None

if result is None:
    print("no result yet")   # no result yet

# Common pattern: function returns None on failure
def safe_divide(a, b):
    if b == 0:
        return None
    return a / b

print(safe_divide(10, 0))   # None
print(safe_divide(10, 2))   # 5.0
```

### type() and isinstance()

Use `type()` for exact type checks. Use `isinstance()` when subclasses should also pass — prefer it in production code and when working with numeric types where `bool` is a subclass of `int`.

```python
x = 42

print(type(x))             # <class 'int'>
print(type(x) == int)      # True — exact match only

print(isinstance(x, int))         # True
print(isinstance(True, int))      # True  — bool IS an int
print(isinstance(True, bool))     # True
print(isinstance(x, (int, float)))# True — check multiple types at once
```

### Type conversion

Explicit conversion is safer than implicit. Know where it fails so you can wrap it in error handling.

```python
print(int("42"))       # 42
print(float("3.14"))   # 3.14
print(str(100))        # '100'
print(bool(0))         # False
print(bool(""))        # False  — empty string is falsy
print(bool("hello"))   # True

# int("3.14") raises ValueError — convert to float first
print(int(float("3.14")))  # 3
```

---

## 2. String Operations

### f-strings — the default way to format strings

f-strings are faster and more readable than `.format()` or `%`. Use them everywhere. Expressions inside `{}` are evaluated at runtime.

```python
name = "Nikhil"
accuracy = 0.9342

print(f"Model trained by {name}")               # Model trained by Nikhil
print(f"Accuracy: {accuracy:.2%}")              # Accuracy: 93.42%
print(f"Pi approx: {22/7:.4f}")                # Pi approx: 3.1429
print(f"{'left':<10}|{'right':>10}")           # left      |     right
print(f"Debug: {name=}")                        # Debug: name='Nikhil'  (Python 3.8+)
```

### .split() and .join()

`.split()` breaks a string into a list. `.join()` reassembles a list into a string. They are inverses of each other and come up constantly in text preprocessing.

```python
sentence = "age,income,churn_label"
columns = sentence.split(",")
print(columns)              # ['age', 'income', 'churn_label']

cleaned = "_".join(columns)
print(cleaned)              # age_income_churn_label

# Split on whitespace (default) strips extra spaces
text = "  hello   world  "
print(text.split())         # ['hello', 'world']
```

### .strip(), .lstrip(), .rstrip()

Strip whitespace (or specified characters) from string edges. Essential when reading messy CSV data where columns may have extra spaces.

```python
raw = "   churn   "
print(raw.strip())          # 'churn'
print(raw.lstrip())         # 'churn   '
print(raw.rstrip())         # '   churn'

# Strip specific characters
path = "///data/file.csv///"
print(path.strip("/"))      # 'data/file.csv'
```

### .replace() and case methods

```python
header = "Customer ID"
snake = header.lower().replace(" ", "_")
print(snake)                # customer_id

print("hello".upper())      # HELLO
print("HELLO".lower())      # hello
print("hello world".title())# Hello World
print("  hi  ".strip())     # hi

# Check string content
print("abc123".isdigit())   # False
print("123".isdigit())      # True
print("abc".isalpha())      # True
```

### String slicing

Strings are sequences — the same slicing rules apply to lists. `s[start:stop:step]`.

```python
s = "data_science"

print(s[0])      # d       — first character
print(s[-1])     # e       — last character
print(s[0:4])    # data    — characters 0,1,2,3
print(s[5:])     # science — from index 5 to end
print(s[:4])     # data    — up to (not including) index 4
print(s[::-1])   # ecneics_atad — reversed
print(s[::2])    # dt_cec — every second character
```

### Checking membership and string methods

```python
text = "Random Forest is an ensemble method"

print("ensemble" in text)          # True
print(text.startswith("Random"))   # True
print(text.endswith("method"))     # True
print(text.count("e"))             # 4
print(text.find("Forest"))         # 7   — index of first match, -1 if not found
print(text.replace("Random", "Gradient"))
# Gradient Forest is an ensemble method
```

---

## 3. Lists

### Creation and indexing

Lists are ordered, mutable, and allow duplicates. They are your default sequential container in Python.

```python
scores = [0.91, 0.87, 0.93, 0.78, 0.95]

print(scores[0])    # 0.91  — first element
print(scores[-1])   # 0.95  — last element
print(scores[1:3])  # [0.87, 0.93]

# Lists can hold mixed types (usually avoid this in data work)
row = [1, "Alice", 29, True]
```

### Common list methods

```python
features = ["age", "income"]

features.append("churn")           # add to end
print(features)                     # ['age', 'income', 'churn']

features.insert(1, "gender")       # insert at index
print(features)                     # ['age', 'gender', 'income', 'churn']

features.remove("gender")          # remove first occurrence by value
print(features)                     # ['age', 'income', 'churn']

popped = features.pop()            # remove and return last
print(popped)                       # churn

features.extend(["region", "plan"])# add multiple items
print(features)                     # ['age', 'income', 'region', 'plan']

print(len(features))               # 4
print(features.index("income"))    # 1
print(features.count("age"))       # 1
```

### Sorting

```python
vals = [3, 1, 4, 1, 5, 9, 2, 6]

vals.sort()                   # in-place, modifies the list
print(vals)                   # [1, 1, 2, 3, 4, 5, 6, 9]

vals.sort(reverse=True)
print(vals)                   # [9, 6, 5, 4, 3, 2, 1, 1]

# sorted() returns a new list — use when you need to keep the original
original = [3, 1, 4, 1, 5]
ranked = sorted(original)
print(original)               # [3, 1, 4, 1, 5]  — unchanged
print(ranked)                 # [1, 1, 3, 4, 5]

# Sort by key
models = [("RandomForest", 0.91), ("XGBoost", 0.93), ("LogReg", 0.87)]
models.sort(key=lambda m: m[1], reverse=True)
print(models)  # [('XGBoost', 0.93), ('RandomForest', 0.91), ('LogReg', 0.87)]
```

### List comprehensions

The most Pythonic way to build a list from another iterable. More readable and faster than a for loop with `.append()`.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

squares = [n**2 for n in numbers]
print(squares)               # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

evens = [n for n in numbers if n % 2 == 0]
print(evens)                 # [2, 4, 6, 8, 10]

# Transformation + filter in one line
normalized = [round((n - 1) / 9, 2) for n in numbers if n > 3]
print(normalized)            # [0.33, 0.44, 0.56, 0.67, 0.78, 0.89, 1.0]
```

### Flattening and copying

```python
nested = [[1, 2], [3, 4], [5, 6]]

# Flatten a list of lists
flat = [x for sublist in nested for x in sublist]
print(flat)                  # [1, 2, 3, 4, 5, 6]

# Shallow copy — avoid mutating the original
original = [1, 2, 3]
copy1 = original[:]          # slice copy
copy2 = original.copy()      # explicit copy
copy3 = list(original)       # constructor copy

copy1.append(99)
print(original)              # [1, 2, 3]  — unaffected
```

---

## 4. Dictionaries

### Creation and basic access

Dictionaries are key-value stores. In Python 3.7+ they preserve insertion order. Use them to represent a single record, a mapping of labels to values, or a lookup table.

```python
model_scores = {
    "logistic_regression": 0.87,
    "random_forest": 0.91,
    "xgboost": 0.93,
}

print(model_scores["xgboost"])            # 0.93
print(len(model_scores))                  # 3
print("svm" in model_scores)             # False  — checks keys
print(list(model_scores.keys()))         # ['logistic_regression', 'random_forest', 'xgboost']
print(list(model_scores.values()))       # [0.87, 0.91, 0.93]
```

### .get() — safe access

Use `.get()` instead of `[]` when the key might not exist. It returns `None` (or a default you specify) rather than raising a `KeyError`.

```python
scores = {"rf": 0.91, "xgb": 0.93}

print(scores["svm"])          # KeyError — crashes
print(scores.get("svm"))      # None — safe
print(scores.get("svm", 0.0)) # 0.0  — custom default
```

### .items(), .update(), .pop()

```python
config = {"lr": 0.01, "epochs": 100, "batch_size": 32}

# Iterate key-value pairs
for param, value in config.items():
    print(f"{param}: {value}")

# Update (merge or overwrite)
config.update({"epochs": 200, "dropout": 0.3})
print(config)
# {'lr': 0.01, 'epochs': 200, 'batch_size': 32, 'dropout': 0.3}

# Remove a key and get its value
removed = config.pop("dropout")
print(removed)    # 0.3
```

### Dict comprehensions

Same idea as list comprehensions. Great for transforming or inverting mappings.

```python
raw = {"Age": 29, "Income": 75000, "Churn": 1}

# Lowercase all keys
cleaned = {k.lower(): v for k, v in raw.items()}
print(cleaned)   # {'age': 29, 'income': 75000, 'churn': 1}

# Invert a mapping
label_map = {0: "no_churn", 1: "churn"}
inverted = {v: k for k, v in label_map.items()}
print(inverted)  # {'no_churn': 0, 'churn': 1}

# Filter by value
high_scores = {model: score for model, score in
               {"rf": 0.91, "lr": 0.78, "xgb": 0.93}.items()
               if score > 0.85}
print(high_scores)  # {'rf': 0.91, 'xgb': 0.93}
```

### defaultdict — skip the key-exists check

`defaultdict` from `collections` automatically creates a default value when you access a missing key. Eliminates boilerplate `if key not in d: d[key] = []` patterns.

```python
from collections import defaultdict

# Group records by category without checking if key exists
records = [("churned", 1), ("retained", 2), ("churned", 3), ("retained", 4)]

grouped = defaultdict(list)
for label, val in records:
    grouped[label].append(val)

print(dict(grouped))   # {'churned': [1, 3], 'retained': [2, 4]}

# Count occurrences
word_count = defaultdict(int)
for word in ["apple", "banana", "apple", "cherry", "banana", "apple"]:
    word_count[word] += 1

print(dict(word_count))  # {'apple': 3, 'banana': 2, 'cherry': 1}
```

### Counter — frequency counts in one line

```python
from collections import Counter

labels = ["cat", "dog", "cat", "bird", "dog", "cat"]
counts = Counter(labels)

print(counts)                      # Counter({'cat': 3, 'dog': 2, 'bird': 1})
print(counts.most_common(2))       # [('cat', 3), ('dog', 2)]
print(counts["cat"])               # 3
print(counts["fish"])              # 0  — no KeyError for missing keys
```

---

## 5. Tuples & Sets

### Tuples — immutable sequences

Use tuples for data that should not change: coordinates, RGB values, function return pairs, dictionary keys. They are faster than lists and signal intent ("this data is fixed").

```python
point = (3.5, 7.2)          # x, y coordinate
rgb = (255, 128, 0)

print(point[0])             # 3.5
# point[0] = 4.0           # TypeError — tuples are immutable

# Tuple unpacking — very common in Python
x, y = point
print(x, y)                 # 3.5  7.2

# Multiple return values use tuples under the hood
def min_max(values):
    return min(values), max(values)   # returns a tuple

low, high = min_max([3, 1, 4, 1, 5, 9])
print(low, high)            # 1  9
```

### Sets — unordered, unique elements

Use sets when you need to eliminate duplicates or perform membership tests on large collections (O(1) lookup vs O(n) for lists). Also use them for set algebra: union, intersection, difference.

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

print(a | b)    # {1, 2, 3, 4, 5, 6, 7, 8} — union
print(a & b)    # {4, 5}                    — intersection
print(a - b)    # {1, 2, 3}                 — difference (in a but not b)
print(a ^ b)    # {1, 2, 3, 6, 7, 8}       — symmetric difference

# Deduplicate a list
raw = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(raw))
print(unique)   # [1, 2, 3, 4]  — order not guaranteed

# Fast membership test
valid_categories = {"electronics", "clothing", "food"}
item = "clothing"
print(item in valid_categories)    # True   — O(1) lookup
```

### When to use which

```python
# Use a tuple when:
# - data is fixed (coordinates, config pairs, dict keys)
# - returning multiple values from a function
# - you want to signal "this shouldn't change"

config = ("localhost", 5432)   # (host, port) — a natural tuple

# Use a set when:
# - you need unique values
# - you need fast membership testing
# - you need set algebra (union, intersection, etc.)

selected_features = {"age", "income", "region"}
required_features = {"age", "credit_score"}

missing = required_features - selected_features
print(missing)    # {'credit_score'}
```

---

## 6. Control Flow

### if / elif / else

```python
score = 0.74

if score >= 0.90:
    grade = "excellent"
elif score >= 0.80:
    grade = "good"
elif score >= 0.70:
    grade = "acceptable"
else:
    grade = "needs improvement"

print(grade)    # acceptable
```

### Ternary expression

For simple conditional assignments, the ternary form is more readable than a full if/else block.

```python
x = 15
label = "odd" if x % 2 != 0 else "even"
print(label)    # odd

# Useful in list comprehensions
scores = [0.91, 0.65, 0.88, 0.72, 0.55]
grades = ["pass" if s >= 0.70 else "fail" for s in scores]
print(grades)   # ['pass', 'fail', 'pass', 'pass', 'fail']
```

### for loops

```python
features = ["age", "income", "region"]

for feature in features:
    print(feature.upper())
# AGE  INCOME  REGION

# Range-based loops
for i in range(5):
    print(i, end=" ")   # 0 1 2 3 4

# range(start, stop, step)
for i in range(0, 10, 2):
    print(i, end=" ")   # 0 2 4 6 8
```

### while, break, continue

```python
# while — use when you don't know the iteration count in advance
attempts = 0
max_attempts = 5

while attempts < max_attempts:
    attempts += 1
    if attempts == 3:
        continue    # skip the rest of this iteration
    if attempts == 4:
        break       # exit the loop entirely
    print(f"attempt {attempts}")
# attempt 1
# attempt 2

# for-else and while-else: else runs only if loop completed without break
for n in [2, 3, 5, 7]:
    if n % 2 == 0 and n != 2:
        print("found even")
        break
else:
    print("no non-two even found")   # this runs
```

### Truthiness — what evaluates to False

Knowing what Python considers falsy saves many unnecessary `== None` or `== []` checks.

```python
# All of these are falsy:
falsy_values = [False, 0, 0.0, "", [], {}, set(), None, (), 0j]

for v in falsy_values:
    if not v:
        print(f"{repr(v):12} is falsy")

# Practical use: check if a list has elements
data = []
if not data:
    print("no data loaded")    # no data loaded

results = [0.91, 0.87]
if results:
    print(f"best: {max(results)}")   # best: 0.91
```

---

## 7. Functions

### def, default arguments, return

Default arguments make functions flexible without requiring every caller to pass every argument. Put defaults at the end of the parameter list.

```python
def evaluate_model(y_true, y_pred, threshold=0.5, verbose=False):
    predictions = [1 if p >= threshold else 0 for p in y_pred]
    correct = sum(t == p for t, p in zip(y_true, predictions))
    accuracy = correct / len(y_true)
    if verbose:
        print(f"Correct: {correct}/{len(y_true)}")
    return accuracy

y_true = [1, 0, 1, 1, 0]
y_pred = [0.9, 0.2, 0.8, 0.4, 0.1]

print(evaluate_model(y_true, y_pred))                    # 0.8
print(evaluate_model(y_true, y_pred, threshold=0.35))    # 0.6
evaluate_model(y_true, y_pred, verbose=True)
# Correct: 4/5
```

### *args and **kwargs

`*args` collects positional arguments into a tuple. `**kwargs` collects keyword arguments into a dict. Use them to write flexible utility functions or wrappers.

```python
def log(*args, **kwargs):
    # args is a tuple, kwargs is a dict
    prefix = kwargs.get("prefix", "INFO")
    message = " ".join(str(a) for a in args)
    print(f"[{prefix}] {message}")

log("Training complete", "epoch=10")
# [INFO] Training complete epoch=10

log("Accuracy", 0.93, prefix="RESULT")
# [RESULT] Accuracy 0.93
```

### lambda — anonymous functions

Use lambda for short, one-off functions — especially as the `key=` argument to `sorted()` or `map()`. If the logic is more than one expression, write a proper `def`.

```python
square = lambda x: x ** 2
print(square(5))    # 25

# Most common use: as a key function
models = [("rf", 0.91), ("lr", 0.78), ("xgb", 0.93)]
ranked = sorted(models, key=lambda m: m[1], reverse=True)
print(ranked)   # [('xgb', 0.93), ('rf', 0.91), ('lr', 0.78)]

# With pandas (preview — covered in pandas cheat sheet)
# df.sort_values(key=lambda col: col.str.lower())
```

### map() and filter()

`map()` applies a function to every element. `filter()` selects elements where the function returns True. Both return lazy iterators — wrap in `list()` to get the result immediately.

```python
values = [1, 4, 9, 16, 25]

roots = list(map(lambda x: x ** 0.5, values))
print(roots)    # [1.0, 2.0, 3.0, 4.0, 5.0]

# filter keeps elements where function returns True
scores = [0.91, 0.65, 0.88, 0.72, 0.55]
passing = list(filter(lambda s: s >= 0.70, scores))
print(passing)  # [0.91, 0.88, 0.72]

# List comprehensions are usually more readable than map/filter
roots_lc = [x ** 0.5 for x in values]
passing_lc = [s for s in scores if s >= 0.70]
```

### Variable scope — LEGB

Python looks up names in this order: Local, Enclosing, Global, Built-in.

```python
threshold = 0.5     # global

def classify(score):
    threshold = 0.7  # local — shadows the global
    return "high" if score >= threshold else "low"

print(classify(0.8))    # high
print(threshold)        # 0.5  — global unchanged

# Use global keyword to modify a global (usually a code smell — prefer return values)
counter = 0
def increment():
    global counter
    counter += 1

increment()
print(counter)   # 1
```

---

## 8. File I/O

### Reading a text file

Always use a `with` block. It guarantees the file is closed even if an exception occurs.

```python
# Write a sample file first
with open("sample.txt", "w") as f:
    f.write("line one\nline two\nline three\n")

# Read entire file as one string
with open("sample.txt", "r") as f:
    content = f.read()
print(content)
# line one
# line two
# line three

# Read line by line — memory-efficient for large files
with open("sample.txt", "r") as f:
    for line in f:
        print(line.strip())   # .strip() removes the trailing newline
```

### Writing and appending

```python
rows = ["Alice,29,1", "Bob,34,0", "Carol,27,1"]

# "w" overwrites the file; "a" appends to it
with open("output.csv", "w") as f:
    f.write("name,age,churn\n")
    for row in rows:
        f.write(row + "\n")

# Append a new row without overwriting
with open("output.csv", "a") as f:
    f.write("Dave,41,0\n")
```

### CSV with the csv module

Use the `csv` module instead of manual string splitting — it handles quoted fields, commas inside values, and different delimiters correctly.

```python
import csv

# Write CSV
data = [
    ["name", "age", "score"],
    ["Alice", 29, 0.91],
    ["Bob", 34, 0.87],
]
with open("data.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(data)

# Read CSV as dicts — column names become keys
with open("data.csv", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(dict(row))
# {'name': 'Alice', 'age': '29', 'score': '0.91'}
# {'name': 'Bob', 'age': '34', 'score': '0.87'}
# Note: all values are strings — cast as needed
```

### Working with file paths

```python
import os
from pathlib import Path  # preferred in Python 3.4+

# pathlib is more readable and cross-platform than os.path
data_dir = Path("data")
csv_file = data_dir / "customers.csv"   # / operator builds paths

print(csv_file.name)        # customers.csv
print(csv_file.stem)        # customers
print(csv_file.suffix)      # .csv
print(csv_file.parent)      # data

# Check existence before reading
if csv_file.exists():
    with open(csv_file) as f:
        pass

# List all CSV files in a directory
for f in Path(".").glob("*.csv"):
    print(f)
```

---

## 9. Error Handling

### try / except / finally

Wrap code that can legitimately fail. Catch specific exceptions — catching bare `Exception` hides bugs.

```python
def load_value(data, key):
    try:
        value = float(data[key])
        return value
    except KeyError:
        print(f"Key '{key}' not found in data")
        return None
    except ValueError:
        print(f"Cannot convert '{data[key]}' to float")
        return None
    finally:
        # Always runs — use for cleanup (closing files, etc.)
        print("load_value finished")

record = {"age": "29", "income": "not_a_number"}

print(load_value(record, "age"))       # 29.0
print(load_value(record, "income"))    # ValueError message, then None
print(load_value(record, "name"))      # KeyError message, then None
```

### Common exception types

```python
# Know these so your except clauses are specific:

# ValueError — right type, wrong value
int("abc")           # ValueError: invalid literal for int()

# TypeError — wrong type for the operation
"5" + 5              # TypeError: can only concatenate str (not "int") to str

# KeyError — dict access with missing key
{}["x"]              # KeyError: 'x'

# IndexError — list/tuple access out of range
[][0]                # IndexError: list index out of range

# AttributeError — attribute doesn't exist on the object
None.split()         # AttributeError: 'NoneType' object has no attribute 'split'

# FileNotFoundError — file doesn't exist
open("ghost.csv")    # FileNotFoundError: [Errno 2] No such file or directory

# ZeroDivisionError
1 / 0                # ZeroDivisionError: division by zero
```

### Raising exceptions

Raise exceptions when your function receives input that violates its contract. This is better than silently returning wrong results.

```python
def train_test_split_check(data, test_size):
    if not 0 < test_size < 1:
        raise ValueError(f"test_size must be between 0 and 1, got {test_size}")
    if len(data) < 2:
        raise ValueError("Need at least 2 samples to split")
    split_idx = int(len(data) * (1 - test_size))
    return data[:split_idx], data[split_idx:]

try:
    train, test = train_test_split_check([1, 2, 3, 4, 5], test_size=1.5)
except ValueError as e:
    print(f"Error: {e}")
# Error: test_size must be between 0 and 1, got 1.5
```

### assert — for development checks

Use `assert` to verify assumptions during development. Do NOT use it for input validation in production — assertions can be disabled with `-O` flag.

```python
def normalize(values):
    assert len(values) > 0, "values cannot be empty"
    min_val = min(values)
    max_val = max(values)
    assert min_val != max_val, "all values are identical — cannot normalize"
    return [(v - min_val) / (max_val - min_val) for v in values]

print(normalize([1, 2, 3, 4, 5]))   # [0.0, 0.25, 0.5, 0.75, 1.0]
```

---

## 10. Itertools & Useful Builtins

### enumerate — loop with index

Use `enumerate` instead of manually tracking an index variable with `i += 1`.

```python
features = ["age", "income", "region", "plan_type"]

for i, feature in enumerate(features):
    print(f"{i}: {feature}")
# 0: age
# 1: income
# 2: region
# 3: plan_type

# Start from a different index
for i, feature in enumerate(features, start=1):
    print(f"{i}. {feature}")
# 1. age  2. income  3. region  4. plan_type
```

### zip — pair up iterables

Combine two or more iterables element-by-element. Stops at the shortest iterable.

```python
models = ["LogReg", "RandomForest", "XGBoost"]
scores = [0.87, 0.91, 0.93]
times  = [0.1, 2.3, 1.8]

for model, score, time in zip(models, scores, times):
    print(f"{model}: acc={score:.2f}, time={time}s")
# LogReg: acc=0.87, time=0.1s
# RandomForest: acc=0.91, time=2.3s
# XGBoost: acc=0.93, time=1.8s

# Unzip — transpose a list of tuples
pairs = [("a", 1), ("b", 2), ("c", 3)]
keys, vals = zip(*pairs)
print(list(keys))   # ['a', 'b', 'c']
print(list(vals))   # [1, 2, 3]
```

### sorted with key, any, all

```python
data = [("Alice", 29, 0.91), ("Bob", 34, 0.78), ("Carol", 27, 0.95)]

# Sort by score descending
by_score = sorted(data, key=lambda row: row[2], reverse=True)
print(by_score[0])   # ('Carol', 27, 0.95)

# any — True if at least one element is truthy
scores = [0.65, 0.72, 0.88]
print(any(s > 0.85 for s in scores))    # True
print(any(s > 0.90 for s in scores))    # False

# all — True only if every element is truthy
print(all(s > 0.60 for s in scores))    # True
print(all(s > 0.70 for s in scores))    # False
```

### itertools.chain — flatten iterables

```python
from itertools import chain

week1 = ["python", "numpy", "pandas"]
week2 = ["sklearn", "matplotlib", "statsmodels"]

all_topics = list(chain(week1, week2))
print(all_topics)
# ['python', 'numpy', 'pandas', 'sklearn', 'matplotlib', 'statsmodels']

# Chain handles any number of iterables
batches = [[1, 2], [3, 4], [5, 6]]
flat = list(chain.from_iterable(batches))
print(flat)   # [1, 2, 3, 4, 5, 6]
```

### itertools.product — cartesian product

Use `product` instead of nested for loops when you want every combination of two or more iterables. Common in hyperparameter grid search.

```python
from itertools import product

learning_rates = [0.01, 0.1]
max_depths = [3, 5, 7]

grid = list(product(learning_rates, max_depths))
print(grid)
# [(0.01, 3), (0.01, 5), (0.01, 7), (0.1, 3), (0.1, 5), (0.1, 7)]

for lr, depth in grid:
    print(f"lr={lr}, depth={depth}")
```

### itertools.combinations and permutations

```python
from itertools import combinations, permutations

features = ["age", "income", "region"]

# All pairs of features (order doesn't matter)
for pair in combinations(features, 2):
    print(pair)
# ('age', 'income')
# ('age', 'region')
# ('income', 'region')

# All orderings of 2 features (order matters)
for perm in permutations(features, 2):
    print(perm)
# ('age', 'income'), ('age', 'region'), ('income', 'age'), ...
```

---

## 11. OOP Basics

### class and __init__

A class bundles data (attributes) and behavior (methods). `__init__` runs automatically when you create an instance. `self` refers to the instance itself — always the first parameter of instance methods.

```python
class ModelEvaluator:
    def __init__(self, model_name, threshold=0.5):
        self.model_name = model_name
        self.threshold = threshold
        self.results = []

    def evaluate(self, y_true, y_pred):
        predictions = [1 if p >= self.threshold else 0 for p in y_pred]
        accuracy = sum(t == p for t, p in zip(y_true, predictions)) / len(y_true)
        self.results.append(accuracy)
        return accuracy

    def best_score(self):
        if not self.results:
            return None
        return max(self.results)

    def __repr__(self):
        # Controls what you see when you print the object
        return f"ModelEvaluator(model='{self.model_name}', threshold={self.threshold})"


evaluator = ModelEvaluator("XGBoost", threshold=0.4)
print(evaluator)   # ModelEvaluator(model='XGBoost', threshold=0.4)

y_true = [1, 0, 1, 1, 0]
y_pred = [0.9, 0.2, 0.8, 0.45, 0.1]
print(evaluator.evaluate(y_true, y_pred))  # 1.0
```

### Inheritance

Subclasses inherit all methods from the parent. Use `super()` to call the parent's `__init__` so you don't duplicate setup code.

```python
class BaseModel:
    def __init__(self, name):
        self.name = name
        self.is_trained = False

    def fit(self, X, y):
        self.is_trained = True
        print(f"{self.name} trained on {len(X)} samples")

    def predict(self, X):
        raise NotImplementedError("Subclasses must implement predict()")


class ThresholdClassifier(BaseModel):
    def __init__(self, name, threshold=0.5):
        super().__init__(name)      # call parent __init__
        self.threshold = threshold

    def predict(self, X):
        if not self.is_trained:
            raise RuntimeError("Call fit() before predict()")
        return [1 if x >= self.threshold else 0 for x in X]


clf = ThresholdClassifier("MyClassifier", threshold=0.6)
clf.fit([0.9, 0.2, 0.8], [1, 0, 1])   # MyClassifier trained on 3 samples
print(clf.predict([0.7, 0.4, 0.9]))    # [1, 0, 1]
```

### @property — controlled attribute access

Use `@property` to make a method look like an attribute. Lets you add validation or computation without changing the calling code.

```python
class Dataset:
    def __init__(self, name, rows, cols):
        self.name = name
        self._rows = rows      # _ prefix signals "internal use"
        self._cols = cols

    @property
    def shape(self):
        return (self._rows, self._cols)

    @property
    def size(self):
        return self._rows * self._cols

    @property
    def rows(self):
        return self._rows

    @rows.setter
    def rows(self, value):
        if value < 0:
            raise ValueError("rows cannot be negative")
        self._rows = value


ds = Dataset("customers", 10000, 15)
print(ds.shape)    # (10000, 15)
print(ds.size)     # 150000
ds.rows = 12000
print(ds.shape)    # (12000, 15)
```

### @classmethod and @staticmethod

`@classmethod` receives the class itself as the first argument — use it for alternative constructors. `@staticmethod` is a plain function attached to the class for organizational purposes.

```python
class Scaler:
    def __init__(self, mean, std):
        self.mean = mean
        self.std = std

    @classmethod
    def from_data(cls, data):
        # Alternative constructor: compute parameters from data
        n = len(data)
        mean = sum(data) / n
        std = (sum((x - mean) ** 2 for x in data) / n) ** 0.5
        return cls(mean, std)

    @staticmethod
    def validate(data):
        return all(isinstance(x, (int, float)) for x in data)

    def transform(self, x):
        return (x - self.mean) / self.std


data = [2.0, 4.0, 6.0, 8.0, 10.0]
scaler = Scaler.from_data(data)   # classmethod — no need to pre-compute
print(f"mean={scaler.mean}, std={scaler.std}")  # mean=6.0, std=2.83...
print(scaler.transform(8.0))                     # ~0.707

print(Scaler.validate(data))          # True
print(Scaler.validate([1, "a", 3]))   # False
```

---

## 12. Comprehensions

### Side-by-side comparison of all four forms

Understanding all four comprehension types lets you pick the right tool for each situation.

```python
numbers = [1, 2, 3, 4, 5]

# List comprehension — returns a list, eager
squares_list = [n**2 for n in numbers]
print(squares_list)              # [1, 4, 9, 16, 25]
print(type(squares_list))        # <class 'list'>

# Dict comprehension — returns a dict
squares_dict = {n: n**2 for n in numbers}
print(squares_dict)              # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
print(type(squares_dict))        # <class 'dict'>

# Set comprehension — returns a set (unique, unordered)
remainders = {n % 3 for n in numbers}
print(remainders)                # {0, 1, 2}
print(type(remainders))          # <class 'set'>

# Generator expression — returns a lazy iterator, NOT a list
squares_gen = (n**2 for n in numbers)
print(squares_gen)               # <generator object ...>
print(next(squares_gen))         # 1  — consumed one at a time
print(list(squares_gen))         # [4, 9, 16, 25]  — rest of the values
```

### List comprehension — with condition

```python
data = [1, -2, 3, -4, 5, -6]

positives = [x for x in data if x > 0]
print(positives)    # [1, 3, 5]

# if/else inside the expression (ternary) — note position differs from filter
clamped = [x if x > 0 else 0 for x in data]
print(clamped)      # [1, 0, 3, 0, 5, 0]

# Nested comprehension — flatten a 2D list
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [val for row in matrix for val in row]
print(flat)         # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Dict comprehension — real-world patterns

```python
# Invert a label-to-index mapping
class_labels = {"cat": 0, "dog": 1, "bird": 2}
index_to_label = {v: k for k, v in class_labels.items()}
print(index_to_label)    # {0: 'cat', 1: 'dog', 2: 'bird'}

# Build a lookup from two lists
features = ["age", "income", "region"]
dtypes   = ["int", "float", "str"]
schema = {f: d for f, d in zip(features, dtypes)}
print(schema)    # {'age': 'int', 'income': 'float', 'region': 'str'}

# Filter dict by value
scores = {"rf": 0.91, "lr": 0.78, "svm": 0.65, "xgb": 0.93}
top_models = {k: v for k, v in scores.items() if v >= 0.85}
print(top_models)    # {'rf': 0.91, 'xgb': 0.93}
```

### Generator expressions — when to use them

Use generator expressions when you don't need all the results at once — they compute values on demand and use far less memory than lists for large datasets.

```python
import sys

large_range = range(1_000_000)

# List uses memory for all 1M items
list_mem = sys.getsizeof([n**2 for n in large_range])

# Generator uses constant memory regardless of size
gen_mem = sys.getsizeof((n**2 for n in large_range))

print(f"List:      {list_mem:,} bytes")    # List:      8,697,464 bytes
print(f"Generator: {gen_mem:,} bytes")     # Generator: 104 bytes

# Pass generator directly to functions that accept iterables
total = sum(n**2 for n in range(1000))    # no list created at all
print(total)    # 332833500

has_large = any(n > 999 for n in range(10000))    # stops at first match
print(has_large)    # True
```

### Set comprehension — deduplication with transformation

```python
# Extract unique domain names from a list of emails
emails = [
    "alice@gmail.com",
    "bob@company.com",
    "carol@gmail.com",
    "dave@university.edu",
    "eve@company.com",
]

domains = {email.split("@")[1] for email in emails}
print(domains)    # {'gmail.com', 'company.com', 'university.edu'}

# Unique first letters of feature names
features = ["age", "annual_income", "balance", "account_age", "balance_ratio"]
initials = {f[0] for f in features}
print(initials)    # {'a', 'b'}
```

---

> [!tip]
> The most important habit: reach for a comprehension when you find yourself writing `result = []` followed by a `for` loop with `.append()`. If the logic fits in one expression, the comprehension is almost always clearer and faster.

> [!warning]
> Nested comprehensions beyond two levels become unreadable. If you need three levels of nesting, write it as explicit loops or break it into helper functions.

> [!success]
> Key patterns to internalize: `enumerate` over manual indexing, `zip` for parallel iteration, `.get()` over `[]` for safe dict access, `with open()` for all file operations, and `isinstance()` over `type() ==` for type checks.
