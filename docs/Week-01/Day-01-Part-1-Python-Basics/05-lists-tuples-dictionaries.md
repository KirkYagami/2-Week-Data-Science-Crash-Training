# Lists, Tuples, and Dictionaries

Before Pandas DataFrames existed, data scientists processed data using lists and dictionaries. Even now, understanding these structures deeply makes you faster and less likely to produce subtle bugs. Every Pandas operation — groupby, apply, merge — returns objects built on these foundations. When Pandas fails or is unavailable, you fall back to these.

---

## Learning Objectives

By the end of this note, you will be able to:

- Create, index, slice, and modify lists with confidence
- Understand the difference between in-place methods and those that return new objects
- Choose between lists and tuples based on mutability and intent
- Use tuple unpacking, including extended unpacking with `*`
- Build, access, and iterate over dictionaries
- Use `.get()` safely to avoid `KeyError` on missing keys
- Write list, dict, and set comprehensions for data transformation
- Use sets for deduplication and O(1) membership testing

---

## Python's Core Collection Types

| Structure | Syntax | Ordered | Mutable | Allows Duplicates |
|-----------|--------|---------|---------|-------------------|
| List | `[1, 2, 3]` | Yes | Yes | Yes |
| Tuple | `(1, 2, 3)` | Yes | No | Yes |
| Dictionary | `{"key": "val"}` | Yes (3.7+) | Yes | Keys: No |
| Set | `{1, 2, 3}` | No | Yes | No |

Choosing the right structure matters. Using a list when you need a set (for membership testing) can make a program 10,000x slower on large datasets.

---

## Lists

A list is an ordered, mutable sequence. It is Python's most-used collection type and the foundation of most data processing loops.

### Creating lists

```python
# Literal syntax
feature_names = ["age", "income", "credit_score", "loan_amount"]
exam_scores = [85, 92, 78, 61, 90]
empty_results = []

# From range — create sequences of numbers
batch_indices = list(range(0, 100, 10))   # [0, 10, 20, ..., 90]

# From another iterable
chars = list("Python")                    # ['P', 'y', 't', 'h', 'o', 'n']

# Nested lists — 2D structures, like a matrix or a table
confusion_matrix = [
    [52, 3, 1],    # true: cat
    [2, 47, 4],    # true: dog
    [0, 1, 38],    # true: bird
]
```

### Indexing and slicing

```python
prices = [10.5, 22.0, 8.75, 31.0, 15.25, 19.99]
#          0     1     2     3      4      5
#         -6    -5    -4    -3     -2     -1

print(prices[0])        # Output: 10.5  — first element
print(prices[-1])       # Output: 19.99 — last element
print(prices[2])        # Output: 8.75

# Slicing [start:stop:step] — stop is exclusive
print(prices[1:4])      # Output: [22.0, 8.75, 31.0]
print(prices[:3])       # Output: [10.5, 22.0, 8.75]  — first 3
print(prices[3:])       # Output: [31.0, 15.25, 19.99] — from index 3 onward
print(prices[::2])      # Output: [10.5, 8.75, 15.25]  — every second element
print(prices[::-1])     # Output: [19.99, 15.25, 31.0, 8.75, 22.0, 10.5] — reversed

# Slicing never raises an IndexError — out-of-range indices are clamped
print(prices[100:200])  # Output: []  — empty, not an error
```

> [!warning]
> Direct indexing with `prices[6]` raises `IndexError` if the index is out of range. But slicing `prices[6:10]` returns `[]` without error. This asymmetry catches many beginners off guard.

### Modifying lists

```python
records = ["Alice", "Bob", "Charlie"]

# Change by index
records[1] = "Barbara"
print(records)    # Output: ['Alice', 'Barbara', 'Charlie']

# append — add one item to the end (O(1))
records.append("Diana")
print(records)    # Output: ['Alice', 'Barbara', 'Charlie', 'Diana']

# extend — add all items from another iterable (O(k) where k is length of added)
records.extend(["Eve", "Frank"])
print(records)    # Output: ['Alice', 'Barbara', 'Charlie', 'Diana', 'Eve', 'Frank']

# insert — add at a specific position (O(n) — shifts everything after)
records.insert(1, "Arun")
print(records)    # Output: ['Alice', 'Arun', 'Barbara', 'Charlie', 'Diana', 'Eve', 'Frank']

# remove — remove first occurrence of a value (raises ValueError if not found)
records.remove("Arun")
print(records)    # Output: ['Alice', 'Barbara', 'Charlie', 'Diana', 'Eve', 'Frank']

# pop — remove and return by index (last element if no index given)
last = records.pop()
print(f"Removed: {last}")    # Output: Removed: Frank
first = records.pop(0)
print(f"Removed: {first}")   # Output: Removed: Alice

# del — remove by index or slice without returning the value
del records[1]               # removes 'Charlie'
print(records)               # Output: ['Barbara', 'Diana', 'Eve']
```

### Essential list methods and functions

```python
scores = [73, 88, 64, 91, 55, 88, 77, 64, 95, 82]

print(len(scores))              # Output: 10
print(min(scores))              # Output: 55
print(max(scores))              # Output: 95
print(sum(scores))              # Output: 777
print(scores.count(88))         # Output: 2  — how many times 88 appears
print(scores.index(91))         # Output: 3  — first index of value 91

# Membership check (O(n) for lists)
print(88 in scores)             # Output: True
print(100 in scores)            # Output: False

# Sorting
sorted_copy = sorted(scores)                 # returns new list, original unchanged
print(sorted_copy)               # Output: [55, 64, 64, 73, 77, 82, 88, 88, 91, 95]
print(scores)                    # Output: [73, 88, ...] — unchanged

scores.sort()                    # sorts IN PLACE, returns None
print(scores)                    # Output: [55, 64, 64, 73, 77, 82, 88, 88, 91, 95]

scores.sort(reverse=True)        # descending in place
print(scores[:3])                # Output: [95, 91, 88]

# Sorting with a custom key
students = [("Alice", 82), ("Bob", 91), ("Charlie", 74), ("Diana", 91)]
ranked = sorted(students, key=lambda s: (-s[1], s[0]))    # score desc, name asc for ties
print(ranked)
# Output: [('Bob', 91), ('Diana', 91), ('Alice', 82), ('Charlie', 74)]
```

> [!warning]
> `list.sort()` modifies the list in place and returns `None`. A common mistake is writing `scores = scores.sort()`, which sets `scores` to `None`. Use `sorted(scores)` when you need a new sorted list and want to preserve the original.

### Copying lists

```python
original = [1, 2, 3, 4, 5]

# These all create REFERENCES — not copies
alias = original
alias.append(6)
print(original)    # Output: [1, 2, 3, 4, 5, 6]  — original changed!

# Shallow copy methods — safe for flat lists
copy1 = original.copy()
copy2 = original[:]
copy3 = list(original)

# Deep copy for nested lists
import copy
matrix = [[1, 2], [3, 4]]
deep = copy.deepcopy(matrix)
deep[0][0] = 99
print(matrix)    # Output: [[1, 2], [3, 4]]  — original unchanged
```

### List comprehensions

List comprehensions are the standard Python way to build transformed or filtered lists. They are more readable than `for` + `append()` and faster at the bytecode level.

```python
# Clean and normalize a column of price strings
raw_prices = ["$10.99", "$25.00", "N/A", "$5.49", "missing", "$15.00"]

valid_prices = [
    float(p[1:])       # strip the '$'
    for p in raw_prices
    if p.startswith("$")    # skip non-price strings
]
print(valid_prices)    # Output: [10.99, 25.0, 5.49, 15.0]

# Create bin labels for a continuous feature
ages = [23, 45, 17, 67, 31, 52, 8, 38]
age_groups = [
    "Child" if a < 18 else "Young Adult" if a < 35 else "Middle Aged" if a < 60 else "Senior"
    for a in ages
]
print(age_groups)
# Output: ['Young Adult', 'Middle Aged', 'Child', 'Senior', 'Young Adult', 'Middle Aged', 'Child', 'Middle Aged']

# Flatten a nested list
batch_results = [[0.82, 0.91], [0.75, 0.88, 0.93], [0.67]]
all_scores = [score for batch in batch_results for score in batch]
print(all_scores)    # Output: [0.82, 0.91, 0.75, 0.88, 0.93, 0.67]
```

---

## Tuples

A tuple is an ordered, **immutable** sequence. Once created, it cannot be changed. This immutability is the point — use tuples when the data should not change.

### When to use tuples instead of lists

- Coordinates: `(latitude, longitude)`
- RGB colors: `(255, 128, 0)`
- Function return values when returning multiple items
- Dictionary keys (lists cannot be dict keys because they are not hashable)
- Any fixed collection where accidental modification would be a bug

```python
# Fixed geographic coordinates — should never change after assignment
berlin_coords = (52.5200, 13.4050)
nyc_coords = (40.7128, -74.0060)

# Tuples can be dict keys; lists cannot
city_data = {
    (52.5200, 13.4050): "Berlin",
    (40.7128, -74.0060): "New York",
}
print(city_data[(52.5200, 13.4050)])    # Output: Berlin

# Attempting to modify raises TypeError
try:
    berlin_coords[0] = 53.0
except TypeError as e:
    print(f"Error: {e}")
# Output: Error: 'tuple' object does not support item assignment
```

### Creating tuples

```python
empty = ()
single = (42,)          # the trailing comma is required for a single-element tuple
pair = (3.14, "pi")
triple = 1, 2, 3        # parentheses are optional!

print(type(single))     # Output: <class 'tuple'>
print(type((42)))       # Output: <class 'int'>  — no comma, just int in parentheses!
```

> [!warning]
> `(42)` is not a tuple — it is the integer `42` in parentheses. To create a single-element tuple you must write `(42,)`. This is a common source of bugs when building tuple collections programmatically.

### Tuple unpacking

```python
# Basic unpacking — assign to multiple names at once
lat, lon = (40.7128, -74.0060)
print(f"Lat: {lat}, Lon: {lon}")
# Output: Lat: 40.7128, Lon: -74.006

# Swap variables without a temp — Python uses tuple packing/unpacking
a, b = 10, 20
a, b = b, a
print(a, b)    # Output: 20 10

# Extended unpacking with *
first, *middle, last = (10, 20, 30, 40, 50)
print(first)     # Output: 10
print(middle)    # Output: [20, 30, 40]
print(last)      # Output: 50

# Skip values you do not need with _
_, score, _ = ("Alice", 92, "A")    # _ is a convention for "ignored"
print(score)    # Output: 92

# Unpack inside a for loop — common with .items() and enumerate()
student_scores = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]
for name, score in student_scores:
    grade = "A" if score >= 90 else "B" if score >= 80 else "C"
    print(f"{name}: {score} → {grade}")
# Output:
# Alice: 85 → B
# Bob: 92 → A
# Charlie: 78 → C
```

### Named tuples — readable fixed records

Named tuples are tuples where each position has a name. They are useful for representing small data records without defining a full class.

```python
from collections import namedtuple

ModelResult = namedtuple("ModelResult", ["model_name", "accuracy", "f1_score", "training_time"])

results = [
    ModelResult("LogisticRegression", 0.842, 0.838, 1.2),
    ModelResult("RandomForest",       0.891, 0.887, 18.4),
    ModelResult("XGBoost",            0.903, 0.899, 45.1),
]

for r in results:
    print(f"{r.model_name:<22} accuracy={r.accuracy:.3f}  f1={r.f1_score:.3f}")
# Output:
# LogisticRegression     accuracy=0.842  f1=0.838
# RandomForest           accuracy=0.891  f1=0.887
# XGBoost                accuracy=0.903  f1=0.899

# Best model by accuracy
best = max(results, key=lambda r: r.accuracy)
print(f"\nBest model: {best.model_name} ({best.accuracy:.1%})")
# Output: Best model: XGBoost (90.3%)
```

---

## Dictionaries

A dictionary maps keys to values. It is Python's implementation of a hash table, which gives O(1) average-case lookup — looking up a key in a million-entry dict takes the same time as looking it up in a ten-entry dict.

### Creating dictionaries

```python
# Literal syntax
customer = {
    "customer_id": 10042,
    "name": "Alice Sharma",
    "tier": "gold",
    "lifetime_value": 12450.0,
    "is_active": True,
}

# From keyword arguments (keys must be valid identifiers)
config = dict(host="localhost", port=5432, database="analytics_db")

# From a list of (key, value) pairs
column_types = dict([("age", int), ("name", str), ("salary", float)])

# From two parallel lists using zip
headers = ["id", "name", "score"]
row_values = [1001, "Alice", 92.5]
record = dict(zip(headers, row_values))
print(record)    # Output: {'id': 1001, 'name': 'Alice', 'score': 92.5}
```

### Accessing values

```python
customer = {"name": "Alice", "age": 30, "tier": "gold"}

# Direct access — raises KeyError if key is missing
print(customer["name"])       # Output: Alice

# .get() — returns None (or a default) if key is missing, never raises
print(customer.get("email"))              # Output: None
print(customer.get("email", "unknown"))   # Output: unknown

# Check before accessing
if "tier" in customer:
    print(f"Tier: {customer['tier']}")    # Output: Tier: gold
```

> [!tip]
> Always use `.get()` when the key might not be present — especially when processing external data (API responses, CSV rows, user input). Direct `dict[key]` access will crash on the first missing key. In production code I have seen pipelines fail silently for days because a dict access raised `KeyError` that was caught too broadly upstream.

### Modifying dictionaries

```python
profile = {"name": "Alice", "age": 30}

# Add or update a single key
profile["email"] = "alice@example.com"    # add new key
profile["age"] = 31                        # update existing key

# Update from another dict
profile.update({"city": "Mumbai", "tier": "gold"})
print(profile)
# Output: {'name': 'Alice', 'age': 31, 'email': 'alice@example.com', 'city': 'Mumbai', 'tier': 'gold'}

# Merge dicts (Python 3.9+)
defaults = {"timeout": 30, "retries": 3, "verbose": False}
overrides = {"timeout": 60, "verbose": True}
config = defaults | overrides    # overrides wins on conflict
print(config)
# Output: {'timeout': 60, 'retries': 3, 'verbose': True}

# Remove a key
del profile["email"]                       # raises KeyError if not present
removed_value = profile.pop("tier", None)  # remove and return; default avoids KeyError
print(removed_value)                       # Output: gold
```

### Iterating over dictionaries

```python
model_scores = {
    "LogisticRegression": 0.842,
    "RandomForest": 0.891,
    "XGBoost": 0.903,
    "NeuralNetwork": 0.887,
}

# Iterate over keys (default)
for model in model_scores:
    print(model)

# Iterate over values
average_score = sum(model_scores.values()) / len(model_scores)
print(f"Average score: {average_score:.3f}")
# Output: Average score: 0.881

# Iterate over key-value pairs — most common pattern
print(f"\n{'Model':<22} {'Score':>8} {'vs Avg':>8}")
print("-" * 40)
for model_name, score in model_scores.items():
    delta = score - average_score
    sign = "+" if delta >= 0 else ""
    print(f"{model_name:<22} {score:>8.3f} {sign + f'{delta:.3f}':>8}")
# Output:
# Model                    Score   vs Avg
# ----------------------------------------
# LogisticRegression       0.842   -0.039
# RandomForest             0.891   +0.010
# XGBoost                  0.903   +0.022
# NeuralNetwork            0.887   +0.006
```

### Dictionary comprehensions

```python
# Map feature names to their missing value counts
features = ["age", "income", "education", "occupation"]
raw_missing = [3, 12, 0, 5]

missing_counts = {feat: count for feat, count in zip(features, raw_missing)}
print(missing_counts)
# Output: {'age': 3, 'income': 12, 'education': 0, 'occupation': 5}

# Filter to only features with missing values
has_missing = {feat: count for feat, count in missing_counts.items() if count > 0}
print(has_missing)
# Output: {'age': 3, 'income': 12, 'occupation': 5}

# Invert a mapping (swap keys and values)
country_codes = {"India": "IN", "Germany": "DE", "Japan": "JP"}
code_to_country = {code: country for country, code in country_codes.items()}
print(code_to_country)
# Output: {'IN': 'India', 'DE': 'Germany', 'JP': 'Japan'}
```

### Nested dictionaries — representing structured records

```python
customer_record = {
    "customer_id": 10042,
    "name": "Alice Sharma",
    "contact": {
        "email": "alice@example.com",
        "phone": "+91-9876543210",
        "address": {
            "city": "Mumbai",
            "state": "Maharashtra",
            "pin": "400001"
        }
    },
    "purchases": [
        {"date": "2024-01-10", "amount": 1250.0, "category": "Electronics"},
        {"date": "2024-02-14", "amount": 340.0, "category": "Clothing"},
    ]
}

# Access nested data
print(customer_record["contact"]["email"])
# Output: alice@example.com

print(customer_record["contact"]["address"]["city"])
# Output: Mumbai

print(customer_record["purchases"][0]["amount"])
# Output: 1250.0

# Safe access for uncertain paths
rating = (customer_record
          .get("profile", {})
          .get("loyalty_score", "Not rated"))
print(rating)    # Output: Not rated
```

---

## Sets

A set is an unordered collection of unique items backed by a hash table. Two primary uses: deduplication and fast membership testing.

### Creating and using sets

```python
# Literal syntax — duplicates are automatically removed
unique_categories = {"Electronics", "Clothing", "Books", "Electronics", "Food"}
print(unique_categories)    # Output: {'Electronics', 'Clothing', 'Books', 'Food'}  (order varies)

# Convert a list to set to remove duplicates
raw_tags = ["python", "sql", "python", "ml", "sql", "deep-learning", "ml"]
unique_tags = set(raw_tags)
print(sorted(unique_tags))    # Output: ['deep-learning', 'ml', 'python', 'sql']
```

### Membership testing — sets vs lists

```python
# List membership: O(n) — scans every element
import time

large_list = list(range(1_000_000))
large_set = set(large_list)

target = 999_999

start = time.perf_counter()
result = target in large_list
list_time = time.perf_counter() - start

start = time.perf_counter()
result = target in large_set
set_time = time.perf_counter() - start

print(f"List lookup: {list_time * 1000:.3f} ms")
print(f"Set lookup:  {set_time * 1000:.3f} ms")
# Typical output:
# List lookup: 12.847 ms
# Set lookup:  0.002 ms
```

> [!tip]
> When you need to check membership repeatedly against a fixed collection, convert it to a set once. A common data science pattern: `valid_ids = set(reference_df["id"])` then `df[df["id"].isin(valid_ids)]`. Pandas `.isin()` does this internally, but if you are writing pure Python, the conversion matters.

### Set operations for data analysis

```python
# Which features are in the training data but not the test data?
train_features = {"age", "income", "credit_score", "employment_years", "loan_amount"}
test_features  = {"age", "income", "credit_score", "marital_status", "loan_amount"}

in_train_not_test = train_features - test_features
print(f"Train only: {in_train_not_test}")     # Output: {'employment_years'}

in_test_not_train = test_features - train_features
print(f"Test only:  {in_test_not_train}")      # Output: {'marital_status'}

common_features = train_features & test_features
print(f"Common:     {common_features}")        # Output: {'age', 'income', 'credit_score', 'loan_amount'}

all_features = train_features | test_features
print(f"All:        {all_features}")           # Output: all 6 features
```

---

## Practical Patterns

### Count frequencies with a dict

```python
# Simulate a column of category labels from a dataset
transaction_categories = [
    "Electronics", "Clothing", "Food", "Electronics", "Books",
    "Clothing", "Electronics", "Food", "Electronics", "Books",
]

# Count occurrences
freq = {}
for category in transaction_categories:
    freq[category] = freq.get(category, 0) + 1

# Sort by frequency
for category, count in sorted(freq.items(), key=lambda x: x[1], reverse=True):
    bar = "#" * count
    print(f"{category:<15} {bar} ({count})")
# Output:
# Electronics     #### (4)
# Clothing        ## (2)
# Food            ## (2)
# Books           ## (2)
```

### Group records by a key

```python
# Group customers by tier — a common pre-aggregation pattern
customers = [
    {"name": "Alice",   "tier": "gold",   "ltv": 12400},
    {"name": "Bob",     "tier": "silver", "ltv": 4200},
    {"name": "Charlie", "tier": "gold",   "ltv": 8900},
    {"name": "Diana",   "tier": "bronze", "ltv": 1100},
    {"name": "Eve",     "tier": "silver", "ltv": 5600},
]

# Build a dict of tier → list of customers
by_tier = {}
for customer in customers:
    tier = customer["tier"]
    if tier not in by_tier:
        by_tier[tier] = []
    by_tier[tier].append(customer["name"])

for tier, names in sorted(by_tier.items()):
    print(f"{tier}: {', '.join(names)}")
# Output:
# bronze: Diana
# gold: Alice, Charlie
# silver: Bob, Eve
```

### Flatten a list of lists

```python
# Common when collecting results from batched processing
batch_predictions = [[0.82, 0.91, 0.75], [0.88, 0.67], [0.93, 0.79, 0.85, 0.71]]

# Comprehension approach
all_predictions = [score for batch in batch_predictions for score in batch]
print(all_predictions)
# Output: [0.82, 0.91, 0.75, 0.88, 0.67, 0.93, 0.79, 0.85, 0.71]

# Alternative: itertools.chain (more efficient for large datasets)
import itertools
all_predictions = list(itertools.chain.from_iterable(batch_predictions))
```

---

## Key Takeaways

> [!success]
> - Lists are ordered, mutable sequences — the default choice for collections of similar items
> - `list.sort()` modifies in place and returns `None`; `sorted(list)` returns a new list — never confuse them
> - Tuples are immutable and hashable — use them for fixed data, function returns, and dict keys
> - Single-element tuples require a trailing comma: `(42,)` not `(42)`
> - Dictionaries give O(1) key lookup — always use `.get(key, default)` when the key might be absent
> - Sets give O(1) membership testing — convert to set before checking `in` on large collections
> - List and dict comprehensions are the Pythonic way to transform and filter collections

---

[[04-functions]] | [[06-practice-problems]]
