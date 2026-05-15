# Python Best Practices

Getting code to run is step one. Writing code that is readable, maintainable, and idiomatic is step two — and it is what determines whether your colleagues can understand your notebooks, whether your own code makes sense six months from now, and whether a technical interviewer views you as someone who writes professional Python.

**Prerequisites:** [[04-exception-handling]]

## Learning Objectives

- Apply PEP 8 naming and formatting conventions consistently
- Write comprehensions that improve readability rather than obscure it
- Add type hints that improve IDE support and serve as lightweight documentation
- Use generators to process large data without memory issues
- Identify and fix the most common Python anti-patterns
- Understand EAFP and when it matters

---

## PEP 8 — The Style Standard

PEP 8 is the official Python style guide. Most Python teams enforce it with automated tools. You do not need to memorize every rule — you need to know the most important ones and let a linter catch the rest.

### Naming Conventions

```python
# Variables and functions: snake_case
customer_tenure = 365
monthly_revenue = 45000.0

def calculate_churn_rate(churned_count: int, total_customers: int) -> float:
    return churned_count / total_customers


# Classes: PascalCase — every word capitalized, no underscores
class CustomerSegment:
    pass

class RandomForestPipeline:
    pass


# Constants: UPPER_SNAKE_CASE — module-level values that never change
MAX_RETRIES = 3
DEFAULT_BATCH_SIZE = 64
API_BASE_URL = "https://api.example.com/v1"


# "Private" by convention: single underscore prefix
_internal_cache = {}  # do not use outside this module

class DataLoader:
    def __init__(self):
        self._connection = None       # internal, do not touch from outside
        self.__session_token = None   # name-mangled — truly hard to access from outside
```

> [!info]
> The single underscore `_name` is a convention meaning "internal use." Python does not enforce it. The double underscore `__name` triggers name mangling — `__secret` in class `MyClass` becomes `_MyClass__secret` externally. This makes accidental overrides in subclasses harder.

### Spacing and Line Length

```python
# Around binary operators: one space each side
total = units_sold * unit_price
churn_rate = churned / total_customers

# No space around = in keyword arguments or default values
def train(learning_rate=0.01, max_epochs=100):
    pass

train(learning_rate=0.05)  # No spaces around the =


# Limit lines to 88-99 characters (most teams use 88 with Black formatter)
# For long expressions, wrap in parentheses:
final_result = (
    first_term
    + second_term
    - adjustment_factor
    * correction_coefficient
)

# Long function signatures
def validate_and_transform(
    records: list[dict],
    required_columns: list[str],
    fill_missing_with: float = 0.0,
) -> list[dict]:
    pass


# Blank lines: 2 between top-level definitions, 1 between methods
class ModelTrainer:

    def __init__(self, model_name: str):
        self.model_name = model_name

    def fit(self, X, y):
        pass

    def evaluate(self, X, y):
        pass


def load_data(filepath: str) -> list:
    pass


def clean_data(records: list) -> list:
    pass
```

---

## Pythonic Code — Writing Python as Python

### List Comprehensions

Comprehensions are the standard Python way to build lists. They are faster than `for` loops with `.append()` because they are optimized at the bytecode level.

```python
employee_salaries = [72000, 55000, 88000, 45000, 120000, 95000]

# Not Pythonic
high_salaries = []
for salary in employee_salaries:
    if salary >= 80000:
        high_salaries.append(salary)

# Pythonic — reads like a sentence
high_salaries = [s for s in employee_salaries if s >= 80000]
print(high_salaries)  # Output: [88000, 120000, 95000]


# Transform and filter in one expression
normalized_high = [s / 100000 for s in employee_salaries if s >= 80000]
print(normalized_high)  # Output: [0.88, 1.2, 0.95]


# Nested: flatten a list of lists
weekly_sales = [[100, 200, 150], [300, 250, 180], [90, 120, 200]]
all_daily_sales = [day for week in weekly_sales for day in week]
print(all_daily_sales)  # Output: [100, 200, 150, 300, 250, 180, 90, 120, 200]
```

> [!warning]
> Comprehensions become unreadable when they span more than two conditions or have complex logic. At that point, a regular `for` loop is more honest. Readability beats cleverness.

### Dict and Set Comprehensions

```python
products = ["laptop", "mouse", "keyboard", "monitor"]
prices = [75000, 600, 1200, 15000]

# Dict comprehension
price_lookup = {product: price for product, price in zip(products, prices)}
print(price_lookup)
# Output: {'laptop': 75000, 'mouse': 600, 'keyboard': 1200, 'monitor': 15000}

# Filter a dict
premium_products = {k: v for k, v in price_lookup.items() if v > 10000}
print(premium_products)
# Output: {'laptop': 75000, 'monitor': 15000}


# Set comprehension — unique values
raw_departments = ["Engineering", "Marketing", "Engineering", "Sales", "Marketing"]
unique_departments = {dept for dept in raw_departments}
print(unique_departments)  # Output: {'Engineering', 'Marketing', 'Sales'}
```

### Iterating Idioms

```python
customer_names = ["Alice", "Bob", "Carol", "Dave"]
customer_scores = [92, 87, 78, 95]

# Use enumerate() — never use range(len(...))
for position, name in enumerate(customer_names, start=1):
    print(f"{position}. {name}")
# Output:
# 1. Alice
# 2. Bob
# 3. Carol
# 4. Dave

# Use zip() to iterate two sequences in parallel
for name, score in zip(customer_names, customer_scores):
    print(f"{name}: {score}")

# Unpack tuples cleanly
coordinates = [(10, 20), (30, 40), (50, 60)]
for x, y in coordinates:
    print(f"x={x}, y={y}")

# Unpack function return values
def compute_stats(values: list) -> tuple:
    return min(values), max(values), sum(values) / len(values)

min_val, max_val, mean_val = compute_stats(customer_scores)
print(f"Min: {min_val}, Max: {max_val}, Mean: {mean_val}")
# Output: Min: 78, Max: 95, Mean: 88.0
```

### `any()` and `all()`

```python
test_results = [True, True, False, True]
scores = [88, 72, 55, 91, 68]

# Check if any score is failing
has_failing = any(score < 60 for score in scores)
print(has_failing)  # Output: True

# Check if all scores pass
all_passing = all(score >= 60 for score in scores)
print(all_passing)  # Output: False

# More practical: validate required fields
required_fields = ["name", "age", "department", "salary"]
record = {"name": "Alice", "age": 31, "department": "Engineering", "salary": 72000}

is_complete = all(field in record for field in required_fields)
print(is_complete)  # Output: True
```

### Membership Testing

```python
# Checking membership in a list is O(n) — slow for large collections
valid_status_codes = [200, 201, 204, 301, 302, 400, 401, 403, 404, 500]
if response_code in valid_status_codes:
    print("Recognized code")

# Use a set for O(1) membership testing — always prefer this when checking many items
success_codes = {200, 201, 204}
if response_code in success_codes:
    handle_success()


# The `not in` operator is clean — no need to negate
if "salary" not in record:
    record["salary"] = 0.0
```

### f-Strings

```python
model_name = "RandomForest"
accuracy = 0.9412
n_features = 25
run_date = "2025-01-15"

# Old style — do not use
print("Model: %s, Accuracy: %.2f" % (model_name, accuracy))
print("Model: {}, Accuracy: {:.2f}".format(model_name, accuracy))

# f-string — current standard
print(f"Model: {model_name}, Accuracy: {accuracy:.2%}")
# Output: Model: RandomForest, Accuracy: 94.12%

print(f"Features: {n_features:,}")         # Output: Features: 25
print(f"Run: {run_date} | {model_name}")   # Output: Run: 2025-01-15 | RandomForest

# Debug format — prints name = value
threshold = 0.85
print(f"{accuracy=:.2f}")  # Output: accuracy=0.94  (Python 3.8+)
```

---

## Type Hints

Type hints do not change how your code runs — Python ignores them at runtime. But they make the code self-documenting, power IDE autocompletion, and let type checkers like `mypy` catch bugs before they reach production.

```python
from typing import Optional


def normalize_salaries(
    records: list[dict],
    target_column: str = "salary",
    scale: float = 1.0,
) -> list[dict]:
    """Normalize salary values to a 0–1 scale."""
    salaries = [r[target_column] for r in records if target_column in r]
    if not salaries:
        return records
    min_s, max_s = min(salaries), max(salaries)
    if min_s == max_s:
        return records
    for record in records:
        if target_column in record:
            record[target_column] = (record[target_column] - min_s) / (max_s - min_s) * scale
    return records


def find_customer(
    customer_id: int,
    records: list[dict],
) -> Optional[dict]:
    """Find a customer by ID. Returns None if not found."""
    for record in records:
        if record.get("id") == customer_id:
            return record
    return None


def segment_by_value(
    records: list[dict],
    value_column: str,
) -> dict[str, list[dict]]:
    """Segment records into high/medium/low based on column values."""
    segments: dict[str, list[dict]] = {"high": [], "medium": [], "low": []}
    for record in records:
        val = record.get(value_column, 0)
        if val >= 0.7:
            segments["high"].append(record)
        elif val >= 0.4:
            segments["medium"].append(record)
        else:
            segments["low"].append(record)
    return segments
```

> [!tip]
> For Python 3.10+, you can write `list[dict]` and `dict[str, list]` directly. For older versions, import from `typing`: `from typing import List, Dict, Optional`. The `Optional[X]` type means "X or None."

---

## Docstrings

Write docstrings for any function that is not completely self-evident. One clear sentence is usually enough. Only add Args/Returns sections when the parameters are not obvious from type hints and names.

```python
def compute_roc_auc(y_true: list[int], y_scores: list[float]) -> float:
    """Return the ROC-AUC score for binary classification predictions."""
    ...


def rolling_mean(values: list[float], window_size: int) -> list[float]:
    """
    Compute rolling mean with the given window size.

    Values before a full window is available use the partial window mean.
    The output list has the same length as the input.
    """
    ...


class FeatureEncoder:
    """
    Encodes categorical features using integer or one-hot encoding.

    Fit on training data, then transform both train and test sets
    to avoid data leakage.
    """

    def fit(self, column: list[str]) -> "FeatureEncoder":
        """Learn the vocabulary from training data."""
        ...

    def transform(self, column: list[str]) -> list[int]:
        """Apply the learned encoding. Unseen values map to -1."""
        ...
```

> [!warning]
> Multi-paragraph docstrings that describe obvious behavior are noise. A docstring should answer: "What does this do and what should I know before calling it?" Not a tutorial.

---

## Generators — Memory-Efficient Iteration

A generator produces values one at a time on demand, instead of building the entire list in memory. For data science with large files, this is essential.

```python
# Regular function — builds the entire list in memory
def load_all_records(filepath: str) -> list[dict]:
    import csv
    with open(filepath, "r", newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))
# If the file has 10 million rows, this uses gigabytes of memory.


# Generator function — yields one record at a time
def stream_records(filepath: str):
    import csv
    with open(filepath, "r", newline="", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            yield row
# Memory footprint is essentially constant, regardless of file size.


# Use it like any iterator
for record in stream_records("large_dataset.csv"):
    process(record)  # only one row in memory at a time


# Generator expression — like a list comprehension but lazy
salaries = [72000, 55000, 88000, 45000, 120000]

# List comprehension — materializes all squared values at once
squared = [s ** 2 for s in salaries]

# Generator expression — computes each value when asked
squared_gen = (s ** 2 for s in salaries)

# sum() works with generators — no need to materialize the list
total_squared = sum(s ** 2 for s in salaries)
print(total_squared)  # Output: 34374750000


# Useful generator: batch processing
def in_batches(iterable, batch_size: int):
    """Yield successive batches of size batch_size."""
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == batch_size:
            yield batch
            batch = []
    if batch:
        yield batch  # yield the final partial batch


data = list(range(1, 11))
for batch in in_batches(data, batch_size=3):
    print(batch)
# Output:
# [1, 2, 3]
# [4, 5, 6]
# [7, 8, 9]
# [10]
```

---

## Anti-Patterns to Avoid

### 1. Bare `except:` or `except Exception:`

```python
# Dangerous — catches everything including Ctrl+C and sys.exit()
try:
    process_record(row)
except:
    pass

# Lazy — catches all exceptions and hides bugs
try:
    process_record(row)
except Exception:
    pass

# Correct — catch what you expect and handle it specifically
try:
    process_record(row)
except (ValueError, KeyError) as e:
    logger.warning(f"Skipping malformed row: {e}")
```

### 2. Mutable Default Arguments

```python
# Bug — the list is created once at function definition time
# and shared across all calls
def add_feature(name: str, feature_list=[]):
    feature_list.append(name)
    return feature_list

print(add_feature("age"))    # Output: ['age']
print(add_feature("income")) # Output: ['age', 'income'] — wrong! reused the same list


# Correct — use None as the sentinel
def add_feature(name: str, feature_list=None) -> list:
    if feature_list is None:
        feature_list = []
    feature_list.append(name)
    return feature_list

print(add_feature("age"))    # Output: ['age']
print(add_feature("income")) # Output: ['income']
```

### 3. Comparing to None with `==`

```python
result = compute()

# Wrong — == calls __eq__, which can be overridden
if result == None:
    handle_missing()

# Correct — is / is not checks identity
if result is None:
    handle_missing()

if result is not None:
    process(result)
```

### 4. String Concatenation in Loops

```python
names = ["Alice", "Bob", "Carol", "Dave", "Eve"]

# Slow — creates a new string object on every iteration: O(n²)
result = ""
for name in names:
    result += name + ", "

# Fast — join builds the final string in one pass: O(n)
result = ", ".join(names)
print(result)  # Output: Alice, Bob, Carol, Dave, Eve
```

### 5. Using a List When a Set Is Needed

```python
seen_ids = []  # checking membership is O(n)

for record in records:
    if record["id"] not in seen_ids:  # O(n) search every iteration
        seen_ids.append(record["id"])

# Use a set — membership check is O(1)
seen_ids = set()

for record in records:
    if record["id"] not in seen_ids:  # O(1)
        seen_ids.add(record["id"])
```

### 6. Global State

```python
# Dangerous — functions that depend on and modify global state are
# hard to test, hard to reason about, and cause subtle bugs in pipelines

model_results = {}  # global

def train_model(data):
    model_results["accuracy"] = 0.92  # mutates global
    model_results["model"] = "rf"


# Better — return results, do not mutate globals
def train_model(data: list) -> dict:
    return {"accuracy": 0.92, "model": "rf"}

results = train_model(training_data)
```

---

## The `dataclasses` Pattern for Configuration

For configuration objects, `@dataclass` beats both dicts and namedtuples for readability:

```python
from dataclasses import dataclass, field
from typing import Optional


@dataclass
class PipelineConfig:
    """Configuration for the training pipeline."""

    input_filepath: str
    output_dir: str
    model_type: str = "random_forest"
    test_size: float = 0.2
    random_state: int = 42
    feature_columns: list = field(default_factory=list)
    target_column: Optional[str] = None
    max_training_rows: Optional[int] = None

    def validate(self) -> None:
        if not 0 < self.test_size < 1:
            raise ValueError(f"test_size must be in (0, 1), got {self.test_size}")
        if self.target_column is None:
            raise ValueError("target_column must be set before training")


config = PipelineConfig(
    input_filepath="data/customers.csv",
    output_dir="output/models",
    feature_columns=["age", "tenure", "monthly_charges"],
    target_column="churn",
)

config.validate()
print(config)
# Output: PipelineConfig(input_filepath='data/customers.csv', ...)
```

---

## Key Takeaways

> [!success]
> - PEP 8: `snake_case` for variables and functions, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants
> - Comprehensions are idiomatic Python for building lists, dicts, and sets — but stop when they get complex
> - Type hints make code self-documenting and power IDE autocompletion
> - Generators process large data without loading it all into memory — essential for real datasets
> - Never use bare `except:`, mutable default arguments, `== None`, or string concatenation in loops
> - Avoid global state — functions should take inputs and return outputs

---

[[04-exception-handling|← Exception Handling]] | [[06-mini-exercises|Mini Exercises →]]
