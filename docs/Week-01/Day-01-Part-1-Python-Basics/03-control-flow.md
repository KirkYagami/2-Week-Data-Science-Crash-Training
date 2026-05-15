# Control Flow

Programs that run straight down from top to bottom are limited to a single, fixed computation. Control flow is what makes programs useful — the ability to make decisions based on data and repeat operations across collections. In data science, you use these patterns to categorize values, filter records, iterate over datasets, and implement algorithms that converge.

---

## Learning Objectives

By the end of this note, you will be able to:

- Write correct `if/elif/else` chains for multi-condition branching
- Understand why Python uses indentation rather than brackets for blocks
- Iterate over lists, strings, ranges, and any iterable with `for`
- Use `enumerate()` and `zip()` for cleaner loop code
- Write list comprehensions with filtering conditions
- Use `while` loops correctly and avoid infinite loops
- Apply `break`, `continue`, and `pass` purposefully
- Recognize when each loop pattern is the right tool

---

## Conditional Statements — `if / elif / else`

Conditionals let your program take different paths depending on data. Every data cleaning step, every feature engineering decision, every model evaluation is built on conditions.

### Basic syntax

```python
if condition:
    # runs when condition is True
elif another_condition:
    # runs when first was False and this is True
else:
    # runs when all conditions above were False
```

Python uses **indentation** (4 spaces) to define code blocks. There are no curly braces. This forces readable code — poorly indented Python is invalid Python.

### Categorizing continuous data

This is one of the most common patterns in feature engineering:

```python
def classify_bmi(bmi):
    """Classify BMI into WHO standard categories."""
    if bmi < 18.5:
        return "Underweight"
    elif bmi < 25.0:
        return "Normal"
    elif bmi < 30.0:
        return "Overweight"
    else:
        return "Obese"

test_values = [16.2, 22.1, 27.8, 35.4]
for bmi in test_values:
    print(f"BMI {bmi:.1f} → {classify_bmi(bmi)}")
# Output:
# BMI 16.2 → Underweight
# BMI 22.1 → Normal
# BMI 27.8 → Overweight
# BMI 35.4 → Obese
```

### Multi-condition logic

```python
credit_score = 720
annual_income = 65000
existing_loans = 1

# Loan approval logic
if credit_score >= 750 and annual_income >= 50000 and existing_loans == 0:
    decision = "Approved — premium rate"
elif credit_score >= 700 and annual_income >= 40000:
    decision = "Approved — standard rate"
elif credit_score >= 650:
    decision = "Manual review required"
else:
    decision = "Declined"

print(f"Credit score {credit_score}: {decision}")
# Output: Credit score 720: Approved — standard rate
```

### Ternary expression (one-line conditional)

```python
# Syntax: value_if_true if condition else value_if_false
exam_score = 73
result = "Pass" if exam_score >= 60 else "Fail"
print(result)    # Output: Pass

# Useful in list comprehensions and assignments — keep it readable
# Good use — simple condition, short values
status = "active" if user_last_login_days < 30 else "inactive"

# Avoid nesting ternaries — they become unreadable fast
# Bad: "A" if x > 90 else "B" if x > 80 else "C" if x > 70 else "F"
```

> [!warning]
> Deep nesting — more than two or three levels of `if` inside `if` — is a code smell. If you find yourself writing four levels deep, the usual fix is to extract inner logic into a named function. Code I have seen in production with six levels of nesting had three hidden bugs that took a week to find.

### Early returns keep logic flat

```python
# Nested version — harder to follow
def get_discount(customer_tier, order_total):
    if customer_tier == "gold":
        if order_total > 500:
            return 0.20
        else:
            return 0.10
    else:
        if order_total > 500:
            return 0.05
        else:
            return 0.0

# Flat version with early returns — easier to scan
def get_discount(customer_tier, order_total):
    if customer_tier == "gold" and order_total > 500:
        return 0.20
    if customer_tier == "gold":
        return 0.10
    if order_total > 500:
        return 0.05
    return 0.0
```

---

## `for` Loops — Iterating Over Sequences

A `for` loop runs its body once for each item in any iterable — lists, strings, ranges, dicts, files, or any object that supports iteration.

### Iterating over a list

```python
feature_names = ["age", "income", "credit_score", "loan_amount"]

for feature in feature_names:
    print(f"Processing feature: {feature}")
# Output:
# Processing feature: age
# Processing feature: income
# Processing feature: credit_score
# Processing feature: loan_amount
```

### `range()` — generating number sequences

```python
# range(stop) — 0 to stop-1
for epoch in range(5):
    print(f"Epoch {epoch}")
# Output: Epoch 0, Epoch 1, Epoch 2, Epoch 3, Epoch 4

# range(start, stop) — start to stop-1
for batch_num in range(1, 6):
    print(f"Processing batch {batch_num}")
# Output: Processing batch 1 through 5

# range(start, stop, step) — with stride
for threshold in range(0, 101, 10):
    print(threshold, end=" ")
# Output: 0 10 20 30 40 50 60 70 80 90 100
```

### `enumerate()` — index and value together

The most common loop pattern for when you need both position and value:

```python
model_metrics = ["accuracy", "precision", "recall", "f1_score"]

for rank, metric in enumerate(model_metrics, start=1):
    print(f"{rank}. {metric}")
# Output:
# 1. accuracy
# 2. precision
# 3. recall
# 4. f1_score

# Versus the manual way — more code, more ways to make mistakes
for i in range(len(model_metrics)):
    print(f"{i + 1}. {model_metrics[i]}")    # same output, uglier
```

> [!tip]
> Always prefer `for item in collection` or `for i, item in enumerate(collection)` over `for i in range(len(collection))`. The `range(len(...))` pattern is a sign you are thinking in another language's idioms.

### `zip()` — parallel iteration

```python
feature_names = ["age", "income", "credit_score"]
feature_importance = [0.32, 0.41, 0.27]
feature_types = ["numeric", "numeric", "numeric"]

for name, importance, dtype in zip(feature_names, feature_importance, feature_types):
    print(f"{name} ({dtype}): {importance:.0%}")
# Output:
# age (numeric): 32%
# income (numeric): 41%
# credit_score (numeric): 27%
```

> [!warning]
> `zip()` stops at the shortest iterable. If your lists have different lengths, the extra elements from the longer list are silently dropped. If you need to catch mismatched lengths, use `zip(a, b, strict=True)` (Python 3.10+) which raises a `ValueError`.

### Iterating over a string

```python
ticker = "AAPL"
for char in ticker:
    print(char, end="-")
# Output: A-A-P-L-
```

### Nested loops

```python
# Build a confusion matrix structure
class_labels = ["cat", "dog", "bird"]
confusion = {}

for true_label in class_labels:
    for predicted_label in class_labels:
        key = f"{true_label}_predicted_as_{predicted_label}"
        confusion[key] = 0   # initialize all cells to 0

print(f"Total cells: {len(confusion)}")    # Output: Total cells: 9
```

---

## List Comprehensions — Pythonic Loop Syntax

List comprehensions create a new list by applying an expression to each item in an iterable, optionally filtering items. They are not just syntactic sugar — they are faster than equivalent `for` loops because they are optimized at the bytecode level.

```python
# Standard for loop approach
squared_odds = []
for n in range(1, 11):
    if n % 2 != 0:
        squared_odds.append(n ** 2)
print(squared_odds)    # Output: [1, 9, 25, 49, 81]

# List comprehension — same result
squared_odds = [n ** 2 for n in range(1, 11) if n % 2 != 0]
print(squared_odds)    # Output: [1, 9, 25, 49, 81]
```

### Data cleaning patterns

```python
# Strip whitespace and normalize case in a list of category labels
raw_categories = ["  Electronics", "CLOTHING ", "food & Beverage", " books  "]
clean_categories = [cat.strip().title() for cat in raw_categories]
print(clean_categories)
# Output: ['Electronics', 'Clothing', 'Food & Beverage', 'Books']

# Parse price strings from CSV
raw_prices = ["$10.99", "$25.00", "$5.49", "N/A", "$15.00"]
prices = [float(p[1:]) for p in raw_prices if p.startswith("$")]
print(prices)    # Output: [10.99, 25.0, 5.49, 15.0]

# Extract column names without prefix
prefixed = ["feat_age", "feat_income", "feat_score", "label"]
features = [col[5:] for col in prefixed if col.startswith("feat_")]
print(features)    # Output: ['age', 'income', 'score']
```

### Dict and set comprehensions

```python
# Dict comprehension — map feature names to their lengths
feature_names = ["age", "income", "credit_score", "loan_amount"]
name_lengths = {name: len(name) for name in feature_names}
print(name_lengths)
# Output: {'age': 3, 'income': 6, 'credit_score': 12, 'loan_amount': 11}

# Filter a dict by value
scores = {"Alice": 92, "Bob": 55, "Charlie": 78, "Diana": 40, "Eve": 88}
passing = {name: score for name, score in scores.items() if score >= 60}
print(passing)
# Output: {'Alice': 92, 'Charlie': 78, 'Eve': 88}

# Set comprehension — unique first characters
words = ["apple", "banana", "avocado", "blueberry", "cherry", "apricot"]
first_letters = {word[0] for word in words}
print(first_letters)    # Output: {'a', 'b', 'c'}  (order may vary)
```

> [!tip]
> Keep comprehensions on one line if they fit. If the expression gets long enough that you need to wrap it, a regular `for` loop with a descriptive loop variable is more readable. Comprehensions are meant to communicate intent, not impress.

---

## `while` Loops — Repeat Until a Condition Changes

Use `while` when you do not know in advance how many iterations you need — the loop continues until a condition becomes false.

```python
# Simulate gradient descent convergence
loss = 100.0
learning_rate = 0.15
iteration = 0
max_iterations = 1000    # always set a safety cap

while loss > 0.001 and iteration < max_iterations:
    loss *= (1 - learning_rate)    # simplified update
    iteration += 1

print(f"Converged at iteration {iteration} with loss {loss:.6f}")
# Output: Converged at iteration 56 with loss 0.000958
```

### Input validation loop

```python
def get_valid_percentage():
    """Keep asking until the user provides a valid percentage."""
    while True:
        raw = input("Enter a percentage (0–100): ")
        try:
            value = float(raw)
            if 0 <= value <= 100:
                return value
            print(f"  {value} is outside the range 0–100. Try again.")
        except ValueError:
            print(f"  '{raw}' is not a number. Try again.")
```

> [!warning]
> Every `while` loop needs a guaranteed exit condition. Either the condition must eventually become false, or there must be a `break` inside. If neither is true, the loop runs forever and locks up your program. Always set a maximum iteration counter when the natural termination depends on external data.

---

## Loop Control — `break`, `continue`, `pass`

### `break` — exit the loop immediately

```python
# Search for the first anomalous reading in a sensor stream
sensor_readings = [1.02, 0.98, 1.01, 8.74, 1.03, 0.97]
anomaly_threshold = 3.0

anomaly_found = None
for i, reading in enumerate(sensor_readings):
    if reading > anomaly_threshold:
        anomaly_found = (i, reading)
        break    # stop as soon as we find the first one

if anomaly_found:
    idx, val = anomaly_found
    print(f"Anomaly at index {idx}: {val}")
# Output: Anomaly at index 3: 8.74
```

### `continue` — skip to the next iteration

```python
# Process records, skipping any with missing values
records = [
    {"name": "Alice", "salary": 72000},
    {"name": "Bob", "salary": None},
    {"name": "Charlie", "salary": 85000},
    {"name": "Diana", "salary": None},
    {"name": "Eve", "salary": 91000},
]

total = 0
valid_count = 0

for record in records:
    if record["salary"] is None:
        print(f"Skipping {record['name']} — missing salary")
        continue    # skip the rest of this iteration
    total += record["salary"]
    valid_count += 1

avg = total / valid_count if valid_count > 0 else 0
print(f"Average salary (valid records only): {avg:,.2f}")
# Output:
# Skipping Bob — missing salary
# Skipping Diana — missing salary
# Average salary (valid records only): 82,666.67
```

### `pass` — do nothing (placeholder)

```python
# Stub out code you will implement later
def train_model(X, y):
    pass    # TODO: implement gradient descent

def evaluate_model(model, X_test, y_test):
    pass    # TODO: compute precision, recall, F1

# pass is also useful in exception handling when you genuinely want to ignore an error
for value in ["10", "abc", "20", "xyz", "30"]:
    try:
        print(int(value))
    except ValueError:
        pass    # silently skip non-numeric values
# Output: 10, 20, 30
```

### `else` on loops — runs when the loop completes without `break`

```python
# Search for a required column in a DataFrame's columns list
required_column = "customer_id"
available_columns = ["name", "age", "email", "purchase_date"]

for col in available_columns:
    if col == required_column:
        print(f"Found '{required_column}'")
        break
else:
    print(f"ERROR: '{required_column}' not found — cannot proceed with join")
# Output: ERROR: 'customer_id' not found — cannot proceed with join
```

> [!info]
> The `for...else` pattern is rarely used, but it cleanly expresses "search for X, do something if not found." The alternative — setting a boolean flag — requires more code and is easier to get wrong.

---

## Real Data Science Control Flow Examples

### Outlier detection and flagging

```python
def flag_outliers(values, n_std=2.0):
    """
    Flag values more than n_std standard deviations from the mean.
    Returns a list of (index, value, is_outlier) tuples.
    """
    n = len(values)
    mean = sum(values) / n
    variance = sum((x - mean) ** 2 for x in values) / n
    std = variance ** 0.5

    results = []
    for i, val in enumerate(values):
        is_outlier = abs(val - mean) > n_std * std
        results.append((i, val, is_outlier))
    return results, mean, std

sensor_data = [10.2, 10.5, 10.1, 10.8, 45.3, 10.3, 9.9, 10.6, -12.1, 10.4]
flagged, mean, std = flag_outliers(sensor_data)

print(f"Mean: {mean:.2f}, Std: {std:.2f}")
for idx, val, is_outlier in flagged:
    marker = "  <-- OUTLIER" if is_outlier else ""
    print(f"  [{idx}] {val:>8.1f}{marker}")
# Output:
# Mean: 12.42, Std: 15.51
#   [0]     10.2
#   [4]     45.3  <-- OUTLIER
#   [8]    -12.1  <-- OUTLIER
```

### Grade distribution with bar chart (text)

```python
exam_scores = [91, 78, 65, 88, 45, 92, 73, 56, 84, 97, 61, 72, 88, 50, 76]

grade_bins = {
    "A (90-100)": 0,
    "B (80-89)":  0,
    "C (70-79)":  0,
    "D (60-69)":  0,
    "F (below 60)": 0,
}

for score in exam_scores:
    if score >= 90:
        grade_bins["A (90-100)"] += 1
    elif score >= 80:
        grade_bins["B (80-89)"] += 1
    elif score >= 70:
        grade_bins["C (70-79)"] += 1
    elif score >= 60:
        grade_bins["D (60-69)"] += 1
    else:
        grade_bins["F (below 60)"] += 1

print(f"\nGrade Distribution (n={len(exam_scores)})")
print("-" * 35)
for grade_label, count in grade_bins.items():
    bar = "#" * count
    print(f"{grade_label:<15} {bar} {count}")
# Output:
# Grade Distribution (n=15)
# -----------------------------------
# A (90-100)      ### 3
# B (80-89)       ### 3
# C (70-79)       #### 4
# D (60-69)       ## 2
# F (below 60)    ### 3
```

---

## Control Flow Patterns in Data Science — Reference

| Scenario | Pattern |
|----------|---------|
| Categorize a continuous variable | `if/elif/else` chain |
| Apply different logic by data type | `if isinstance(val, int)` |
| Process every row in a dataset | `for row in records` |
| Find the first matching record | `for` with `break` |
| Skip rows with missing values | `if val is None: continue` |
| Retry until convergence | `while error > threshold` |
| Build a transformed column | List comprehension |
| Group items by key | Dict comprehension or `defaultdict` |

---

## Key Takeaways

> [!success]
> - Indentation defines blocks in Python — 4 spaces, always consistent
> - Use `if/elif/else` for decisions; keep conditions ordered from most specific to least specific
> - Prefer `for item in collection` over index-based loops; use `enumerate()` when you need the index
> - List comprehensions are faster and more readable than equivalent `for` + `append()` patterns
> - `while` loops need a guaranteed exit — always add a max-iteration safeguard
> - `break` exits a loop early; `continue` skips the current iteration; `pass` is a no-op placeholder

---

[[02-variables-and-data-types]] | [[04-functions]]
