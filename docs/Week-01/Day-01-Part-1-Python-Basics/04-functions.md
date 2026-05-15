# Functions

The moment your data science code grows beyond a single notebook cell, you need functions. Without them, every change to your logic requires hunting through hundreds of lines of duplicated code, hoping you caught every copy. With well-designed functions, you change one place and the improvement propagates everywhere. Functions are also the unit of testing — you cannot write reliable tests for a 300-line script, but you can write them for a 15-line function.

---

## Learning Objectives

By the end of this note, you will be able to:

- Define and call functions with positional, default, `*args`, and `**kwargs` parameters
- Use early returns to keep function logic flat and readable
- Return multiple values and unpack them at the call site
- Write concise docstrings that document intent and parameters
- Use lambda functions with `map()`, `filter()`, and `sorted()`
- Explain Python's LEGB scope rules and avoid global variable bugs
- Treat functions as first-class values and pass them as arguments

---

## What a Function Is and Why It Matters

A function is a named, callable block of code. When you call it, Python jumps to that block, executes it with whatever arguments you provided, and returns a result.

The most important benefit of functions is not code reuse — it is **making code easier to reason about**. A function with a clear name tells you what it does without you needing to read its implementation. `calculate_discount(price, tier)` is self-documenting. The same calculation written inline three times is not.

```python
# Without functions — three copies of the same logic
order1_discount = 1500 * 0.10 if 1500 > 1000 else 1500 * 0.05
order2_discount = 250 * 0.10 if 250 > 1000 else 250 * 0.05
order3_discount = 3200 * 0.10 if 3200 > 1000 else 3200 * 0.05

# With a function — change the threshold once, it applies everywhere
def calculate_discount(order_total, high_value_threshold=1000):
    """Return the discount amount for an order."""
    rate = 0.10 if order_total > high_value_threshold else 0.05
    return order_total * rate

order1_discount = calculate_discount(1500)
order2_discount = calculate_discount(250)
order3_discount = calculate_discount(3200)
```

---

## Defining and Calling Functions

### Basic syntax

```python
def function_name(parameter1, parameter2):
    """One-line description of what this function does."""
    # function body
    return result
```

### Your first functions

```python
def greet_customer(name, is_premium=False):
    """Return a greeting tailored to the customer's tier."""
    if is_premium:
        return f"Welcome back, {name}! Your exclusive benefits are ready."
    return f"Hello, {name}! How can we help you today?"

print(greet_customer("Alice"))
# Output: Hello, Alice! How can we help you today?

print(greet_customer("Bob", is_premium=True))
# Output: Welcome back, Bob! Your exclusive benefits are ready.
```

### Functions without a return statement

A function that has no `return` statement implicitly returns `None`. This is intentional for functions that perform actions (printing, writing to a file) rather than computing a value.

```python
def print_section_header(title, width=60):
    """Print a formatted section header."""
    print(f"\n{'=' * width}")
    print(f"  {title}")
    print(f"{'=' * width}")

print_section_header("Model Evaluation Results")
# Output:
# ============================================================
#   Model Evaluation Results
# ============================================================

result = print_section_header("test")
print(result)    # Output: None
```

---

## Parameters and Arguments

### Required (positional) parameters

```python
def euclidean_distance(x1, y1, x2, y2):
    """Compute straight-line distance between two 2D points."""
    return ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5

dist = euclidean_distance(0, 0, 3, 4)
print(dist)    # Output: 5.0  — classic 3-4-5 triangle
```

### Default parameters

Default parameters make arguments optional. Place them after all required parameters.

```python
def normalize(value, min_val=0.0, max_val=1.0):
    """Scale a value to the range [min_val, max_val]."""
    return (value - min_val) / (max_val - min_val)

# With defaults
print(normalize(0.5))              # Output: 0.5  — already in [0, 1]

# Overriding defaults
print(normalize(75, min_val=0, max_val=100))    # Output: 0.75
```

> [!warning]
> **Never use a mutable object as a default parameter.** This is one of Python's most notorious bugs. The default is created once when the function is defined and shared across all calls.
> ```python
> # Bug — the list persists between calls!
> def add_record(record, records=[]):
>     records.append(record)
>     return records
>
> print(add_record("Alice"))    # Output: ['Alice']
> print(add_record("Bob"))      # Output: ['Alice', 'Bob']  — unexpected!
>
> # Fix — use None and create the list inside the function
> def add_record(record, records=None):
>     if records is None:
>         records = []
>     records.append(record)
>     return records
>
> print(add_record("Alice"))    # Output: ['Alice']
> print(add_record("Bob"))      # Output: ['Bob']  — correct
> ```

### Keyword arguments

You can pass any argument by name, which removes dependence on order:

```python
def create_model_config(algorithm, learning_rate=0.01, max_epochs=100, random_seed=42):
    """Return a model configuration dictionary."""
    return {
        "algorithm": algorithm,
        "learning_rate": learning_rate,
        "max_epochs": max_epochs,
        "random_seed": random_seed,
    }

# Passing by keyword — order does not matter
config = create_model_config(
    max_epochs=500,
    algorithm="gradient_boosting",
    learning_rate=0.05,
)
print(config)
# Output: {'algorithm': 'gradient_boosting', 'learning_rate': 0.05, 'max_epochs': 500, 'random_seed': 42}
```

### `*args` — variable-length positional arguments

```python
def weighted_average(*values_and_weights):
    """
    Compute a weighted average.
    Alternating arguments: value, weight, value, weight, ...
    """
    total_weighted = 0
    total_weight = 0
    for i in range(0, len(values_and_weights), 2):
        value = values_and_weights[i]
        weight = values_and_weights[i + 1]
        total_weighted += value * weight
        total_weight += weight
    return total_weighted / total_weight

# Exam score weighted average: score, credit_hours
gpa = weighted_average(85, 3, 92, 4, 78, 2)
print(f"Weighted average: {gpa:.1f}")    # Output: Weighted average: 86.2
```

More commonly, `*args` is used to accept a variable number of similar items:

```python
def compute_statistics(*values):
    """Compute mean and range of any number of numeric inputs."""
    if not values:
        return None
    n = len(values)
    mean = sum(values) / n
    return {"count": n, "mean": round(mean, 4), "min": min(values), "max": max(values)}

print(compute_statistics(10, 20, 30))
# Output: {'count': 3, 'mean': 20.0, 'min': 10, 'max': 30}

print(compute_statistics(5))
# Output: {'count': 1, 'mean': 5.0, 'min': 5, 'max': 5}
```

> [!info]
> Inside the function, `*args` is a tuple, not a list. You can iterate over it and index into it, but you cannot append to it. If you need to modify the collection, convert it first: `items = list(args)`.

### `**kwargs` — variable-length keyword arguments

```python
def log_event(event_type, **metadata):
    """Log a structured event with arbitrary metadata fields."""
    import datetime
    timestamp = datetime.datetime.now().isoformat()
    log_entry = {
        "timestamp": timestamp,
        "event_type": event_type,
        **metadata    # merge metadata into the log entry
    }
    for key, value in log_entry.items():
        print(f"  {key}: {value}")
    print()

log_event("model_trained",
          model_name="XGBoost",
          accuracy=0.924,
          training_rows=15000,
          duration_seconds=47.3)
# Output:
#   timestamp: 2024-01-15T14:32:01.123456
#   event_type: model_trained
#   model_name: XGBoost
#   accuracy: 0.924
#   training_rows: 15000
#   duration_seconds: 47.3
```

### Combining all parameter types

The order must be: required → default → `*args` → keyword-only → `**kwargs`.

```python
def pipeline_step(step_name, verbose=False, *transformations, **config):
    """Simulate a data pipeline step."""
    print(f"Running step: {step_name}")
    if verbose:
        print(f"  Transformations: {transformations}")
        print(f"  Config: {config}")

pipeline_step("normalize", True, "scale", "clip", min_val=0, max_val=1)
# Output:
# Running step: normalize
#   Transformations: ('scale', 'clip')
#   Config: {'min_val': 0, 'max_val': 1}
```

---

## Return Values

### Returning multiple values

Python functions can return multiple values as a tuple. Unpack at the call site for clean code.

```python
def compute_train_test_split(data, test_fraction=0.2, random_seed=None):
    """Split data into training and test sets."""
    import random
    if random_seed is not None:
        random.seed(random_seed)

    shuffled = data[:]
    random.shuffle(shuffled)

    split_point = int(len(shuffled) * (1 - test_fraction))
    train_data = shuffled[:split_point]
    test_data = shuffled[split_point:]
    return train_data, test_data    # returns a tuple

records = list(range(100))
train, test = compute_train_test_split(records, test_fraction=0.2, random_seed=42)
print(f"Train size: {len(train)}")    # Output: Train size: 80
print(f"Test size:  {len(test)}")     # Output: Test size:  20
```

### Returning dictionaries for named results

When you need to return more than 2-3 values, a dictionary is clearer than positional tuple unpacking:

```python
def evaluate_classifier(y_true, y_pred):
    """Compute classification metrics."""
    n = len(y_true)
    correct = sum(t == p for t, p in zip(y_true, y_pred))
    true_pos = sum(t == 1 and p == 1 for t, p in zip(y_true, y_pred))
    false_pos = sum(t == 0 and p == 1 for t, p in zip(y_true, y_pred))
    false_neg = sum(t == 1 and p == 0 for t, p in zip(y_true, y_pred))

    precision = true_pos / (true_pos + false_pos) if (true_pos + false_pos) > 0 else 0
    recall = true_pos / (true_pos + false_neg) if (true_pos + false_neg) > 0 else 0
    f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0

    return {
        "accuracy": correct / n,
        "precision": precision,
        "recall": recall,
        "f1_score": f1,
        "n_samples": n,
    }

y_true = [1, 0, 1, 1, 0, 1, 0, 0, 1, 1]
y_pred = [1, 0, 1, 0, 0, 1, 1, 0, 1, 0]

metrics = evaluate_classifier(y_true, y_pred)
for metric, value in metrics.items():
    print(f"  {metric:<12}: {value:.4f}")
# Output:
#   accuracy    : 0.7000
#   precision   : 0.8000
#   recall      : 0.6667
#   f1_score    : 0.7273
#   n_samples   : 10.0000
```

---

## Docstrings

A docstring is the string immediately after the `def` line. It is the official way to document a function and is accessible at runtime via `help(function_name)` or `function_name.__doc__`.

```python
def clean_text_column(text, lowercase=True, strip_punctuation=False):
    """
    Clean a text string for NLP preprocessing.

    Args:
        text (str): The raw input string.
        lowercase (bool): Convert to lowercase. Default True.
        strip_punctuation (bool): Remove all punctuation characters. Default False.

    Returns:
        str: The cleaned string, with leading/trailing whitespace removed.

    Raises:
        TypeError: If text is not a string.

    Examples:
        >>> clean_text_column("Hello, World!")
        'hello, world!'
        >>> clean_text_column("Hello, World!", strip_punctuation=True)
        'hello world'
    """
    if not isinstance(text, str):
        raise TypeError(f"Expected str, got {type(text).__name__}")

    result = text.strip()
    if lowercase:
        result = result.lower()
    if strip_punctuation:
        import string
        result = result.translate(str.maketrans("", "", string.punctuation))
    return result

# Test it
print(clean_text_column("  Hello, World!  "))
# Output: hello, world!

print(clean_text_column("  Hello, World!  ", strip_punctuation=True))
# Output: hello world
```

> [!tip]
> Write the docstring before writing the implementation. Describing what a function should do forces you to think clearly about its interface — what it accepts, what it returns, and what can go wrong. This makes the implementation much easier to write.

---

## Lambda Functions

A lambda is an anonymous function defined in a single expression. Use them when you need a small function in one place and naming it would add more ceremony than clarity.

```python
# Regular function
def square(x):
    return x ** 2

# Lambda equivalent
square = lambda x: x ** 2
print(square(9))    # Output: 81
```

Lambdas are most useful as arguments to higher-order functions:

```python
# sorted() — custom sort key
leaderboard = [
    {"name": "Alice", "score": 1450, "level": 8},
    {"name": "Bob",   "score": 1820, "level": 7},
    {"name": "Charlie", "score": 1820, "level": 9},
]

# Sort by score descending, then by level descending for ties
ranked = sorted(leaderboard,
                key=lambda player: (player["score"], player["level"]),
                reverse=True)
for rank, player in enumerate(ranked, 1):
    print(f"{rank}. {player['name']} — Score: {player['score']}, Level: {player['level']}")
# Output:
# 1. Charlie — Score: 1820, Level: 9
# 2. Bob — Score: 1820, Level: 7
# 3. Alice — Score: 1450, Level: 8

# map() — apply a transformation to every element
revenues = [85000, 120000, 45000, 200000, 73000]
in_millions = list(map(lambda r: round(r / 1e6, 3), revenues))
print(in_millions)    # Output: [0.085, 0.12, 0.045, 0.2, 0.073]

# filter() — keep elements that satisfy a condition
high_earners = list(filter(lambda r: r > 100000, revenues))
print(high_earners)    # Output: [120000, 200000]
```

### Lambdas in Pandas (extremely common)

```python
import pandas as pd

df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "salary": [72000, 95000, 130000],
    "years_experience": [3, 7, 12],
})

# Create a new column using a lambda
df["salary_per_year_exp"] = df.apply(
    lambda row: round(row["salary"] / row["years_experience"]),
    axis=1
)

print(df)
# Output:
#       name  salary  years_experience  salary_per_year_exp
# 0    Alice   72000                 3                24000
# 1      Bob   95000                 7                13571
# 2  Charlie  130000                12                10833
```

> [!warning]
> Lambdas cannot contain statements (only expressions), cannot have docstrings, and produce unhelpful error messages because they are anonymous. If a lambda is getting complex — multiple conditions, multiple lines — refactor it into a named function. The name alone communicates intent that the lambda cannot.

---

## Variable Scope — The LEGB Rule

Python looks up variable names in this order:

1. **L**ocal — inside the current function
2. **E**nclosing — inside any function that wraps the current one
3. **G**lobal — at the module level (top of the file)
4. **B**uilt-in — Python's own built-in names (`len`, `print`, etc.)

```python
threshold = 0.5    # Global

def apply_threshold(scores):
    # threshold here refers to the global variable
    # Python found it by walking up: Local (not found) → Global (found)
    return [s for s in scores if s > threshold]

print(apply_threshold([0.3, 0.7, 0.1, 0.9]))
# Output: [0.7, 0.9]

def apply_custom_threshold(scores, threshold=0.8):
    # threshold here is Local — shadows the global
    return [s for s in scores if s > threshold]

print(apply_custom_threshold([0.3, 0.7, 0.1, 0.9]))
# Output: [0.9]
```

### The `global` keyword — use it sparingly

```python
model_call_count = 0    # module-level counter

def predict(features):
    global model_call_count    # explicitly modify the global
    model_call_count += 1
    # ... prediction logic ...
    return 0

predict([1, 2, 3])
predict([4, 5, 6])
print(model_call_count)    # Output: 2
```

> [!warning]
> Avoid `global` in production code. Functions that modify global state are hard to test, hard to reason about, and create subtle bugs when the same module is used concurrently. The better pattern is to pass state as arguments and return updated state. If you need a counter shared across calls, use a class with instance state.

---

## Functions as First-Class Values

Functions in Python are objects. You can store them in variables, put them in lists, pass them to other functions, and return them from functions. This is what makes patterns like callbacks, pipelines, and decorators possible.

```python
# Storing functions in a data structure
def min_max_normalize(values):
    low, high = min(values), max(values)
    return [(v - low) / (high - low) for v in values]

def z_score_normalize(values):
    n = len(values)
    mean = sum(values) / n
    std = (sum((v - mean) ** 2 for v in values) / n) ** 0.5
    return [(v - mean) / std for v in values]

def log_transform(values):
    import math
    return [math.log(v + 1) for v in values]    # +1 to handle zeros

normalizers = {
    "min_max": min_max_normalize,
    "z_score": z_score_normalize,
    "log":     log_transform,
}

# Select and apply at runtime
raw_data = [2, 8, 1, 15, 4, 23, 7]
method = "z_score"

normalized = normalizers[method](raw_data)
print(f"Method: {method}")
print([round(v, 3) for v in normalized])
# Output:
# Method: z_score
# [-0.638, 0.181, -0.819, 0.92, -0.457, 1.839, 0.0]   (approximate)
```

### Building a simple transformation pipeline

```python
def make_pipeline(*steps):
    """
    Create a function that applies a sequence of transformations in order.
    Each step is a function that takes a list and returns a list.
    """
    def pipeline(data):
        result = data
        for step in steps:
            result = step(result)
        return result
    return pipeline

def remove_negatives(values):
    return [v for v in values if v >= 0]

def remove_outliers(values, cap=100):
    return [v for v in values if v <= cap]

def round_values(values, decimals=2):
    return [round(v, decimals) for v in values]

# Build the pipeline
clean = make_pipeline(remove_negatives, remove_outliers, round_values)

messy_data = [-5, 2.555, 3.141, 150, 8.9, -1, 75, 0.1]
print(clean(messy_data))
# Output: [2.56, 3.14, 8.9, 75, 0.1]
```

---

## Real Data Science Function Patterns

### Safe data extraction from nested structures

```python
def safe_get(obj, *keys, default=None):
    """
    Safely traverse a nested dict/list using a chain of keys.
    Returns default if any key is missing or the type is wrong.
    """
    current = obj
    for key in keys:
        try:
            current = current[key]
        except (KeyError, IndexError, TypeError):
            return default
    return current

# Nested API response (common in real-world data ingestion)
api_response = {
    "status": "success",
    "data": {
        "user": {
            "id": 1001,
            "profile": {
                "name": "Alice",
                "metrics": {"sessions": 47, "revenue": 1250.0}
            }
        }
    }
}

print(safe_get(api_response, "data", "user", "profile", "name"))
# Output: Alice

print(safe_get(api_response, "data", "user", "profile", "age"))
# Output: None  — key does not exist

print(safe_get(api_response, "data", "user", "profile", "metrics", "revenue"))
# Output: 1250.0

print(safe_get(api_response, "data", "orders", 0, "total", default=0.0))
# Output: 0.0  — path does not exist
```

### Batch processing with progress tracking

```python
def process_in_batches(records, batch_size, process_fn, verbose=True):
    """
    Apply process_fn to records in batches of batch_size.
    Returns all results combined.
    """
    results = []
    total = len(records)
    num_batches = (total + batch_size - 1) // batch_size    # ceiling division

    for batch_num in range(num_batches):
        start = batch_num * batch_size
        end = min(start + batch_size, total)
        batch = records[start:end]

        batch_results = process_fn(batch)
        results.extend(batch_results)

        if verbose:
            progress = end / total * 100
            print(f"  Batch {batch_num + 1}/{num_batches} — {end}/{total} records ({progress:.0f}%)")

    return results

# Simulate processing
sample_records = list(range(25))
processed = process_in_batches(
    records=sample_records,
    batch_size=8,
    process_fn=lambda batch: [x * 2 for x in batch]
)
print(f"Processed {len(processed)} records")
# Output:
#   Batch 1/4 — 8/25 records (32%)
#   Batch 2/4 — 16/25 records (64%)
#   Batch 3/4 — 24/25 records (96%)
#   Batch 4/4 — 25/25 records (100%)
# Processed 25 records
```

---

## Key Takeaways

> [!success]
> - Functions make code reusable, testable, and easier to reason about — write one whenever you find yourself repeating logic
> - Use default parameters for optional behavior; never use mutable objects (lists, dicts) as defaults
> - `*args` collects extra positional arguments as a tuple; `**kwargs` collects keyword arguments as a dict
> - Write docstrings that describe what the function does, its parameters, and what it returns
> - Lambda functions are for short, throwaway expressions — use named functions for anything complex
> - Python's LEGB scope rule determines where a name is looked up; avoid `global` in production code
> - Functions are first-class objects — you can store, pass, and return them, which enables clean pipeline and strategy patterns

---

[[03-control-flow]] | [[05-lists-tuples-dictionaries]]
