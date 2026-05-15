# Mini Exercises: Advanced Python

Work through these in order. Each one is small enough to finish in a few minutes, but each one practices a pattern you will reuse constantly in real data science work.

**Prerequisites:** [[01-oop-basics]], [[02-file-handling]], [[03-modules-and-packages]], [[04-exception-handling]], [[05-python-best-practices]]

**Time:** ~30 minutes for warm-up and main problems, ~45 minutes with stretch goals.

> [!tip]
> Type the code yourself — do not copy-paste. The friction of typing is part of learning. Run each solution before moving on.

---

## Exercise 1 — Student Grade Tracker (OOP Warm-up)

**Topic:** Classes, instance methods, `__repr__`

Create a `Student` class that:

- Stores `name` and a list of `scores`
- Has an `average()` method returning the mean score
- Has a `grade()` method returning `"A"` / `"B"` / `"C"` / `"D"` / `"F"` based on the average
- Has a `__repr__` that shows the student's name and grade

### Starter Code

```python
class Student:
    def __init__(self, name: str, scores: list[float]):
        self.name = name
        self.scores = scores

    def average(self) -> float:
        # your code here
        pass

    def grade(self) -> str:
        # your code here
        pass

    def __repr__(self) -> str:
        # your code here
        pass


alice = Student("Alice", [88, 92, 79, 95])
bob = Student("Bob", [55, 62, 48, 70])

print(alice.average())
print(alice.grade())
print(repr(alice))
```

### Expected Output

```
88.5
B
Student('Alice', grade=B)
```

### Stretch Goal

Add a `top_score()` and `needs_improvement()` method (returns `True` if any individual score is below 60). Test with `bob`.

---

## Exercise 2 — Safe Data Parser (Exception Handling Warm-up)

**Topic:** `try / except`, returning `None` on failure

Write a function `safe_parse_record(raw: dict) -> dict | None` that:

- Converts `name` to a stripped, title-cased string
- Converts `age` to `int`
- Converts `salary` to `float`
- Returns `None` if any field is missing or cannot be converted
- Does not crash on any input

### Starter Code

```python
def safe_parse_record(raw: dict) -> dict | None:
    # your code here
    pass


records = [
    {"name": "  alice  ", "age": "31", "salary": "72000"},
    {"name": "BOB", "age": "not available", "salary": "55000"},
    {"name": "carol", "salary": "88000"},  # missing age
    {"name": "", "age": "28", "salary": "-500"},  # empty name
]

for rec in records:
    result = safe_parse_record(rec)
    print(result)
```

### Expected Output

```
{'name': 'Alice', 'age': 31, 'salary': 72000.0}
None
None
{'name': '', 'age': 28, 'salary': -500.0}
```

> [!info]
> The empty name case still parses successfully — the function does not validate business rules, it only converts types. Validation is a separate concern.

### Stretch Goal

Modify the function to also return an error reason when it fails:

```python
result, error = safe_parse_record(rec)
# ("{'name': ...}", None) on success
# (None, "Cannot convert age: invalid literal...") on failure
```

---

## Exercise 3 — CSV Revenue Calculator (File Handling)

**Topic:** `csv.DictReader`, type conversion, accumulation

Create `sales.csv` with this content:

```csv
product,quantity,unit_price
Laptop,12,75000
Monitor,8,15000
Keyboard,25,1200
Mouse,40,600
Headset,invalid,3500
,15,2000
```

Write a script that:

1. Reads the CSV
2. Computes `revenue = quantity * unit_price` for each valid row
3. Prints each product and its revenue
4. Skips invalid rows with a warning message
5. Prints the total revenue at the end

### Expected Output

```
Laptop: 900,000
Monitor: 120,000
Keyboard: 30,000
Mouse: 24,000
Warning: Skipping row — cannot parse 'quantity': invalid literal for int() with base 10: 'invalid'
Warning: Skipping row — cannot parse 'product': empty product name
Total Revenue: 1,074,000
```

### Stretch Goal

Save the valid rows (with a `revenue` column added) to `sales_with_revenue.csv`.

---

## Exercise 4 — JSON Config Loader (File Handling + Exception Handling)

**Topic:** `json.load`, `FileNotFoundError`, `json.JSONDecodeError`

Write a function `load_model_config(path: str) -> dict` that:

- Loads and returns a JSON config file
- Raises a clear `FileNotFoundError` if the file does not exist
- Raises a clear `ValueError` if the JSON is malformed
- Validates that `"model_type"` and `"target_column"` keys are present, raising `KeyError` if either is missing

### Config File

Create `model_config.json`:

```json
{
  "model_type": "random_forest",
  "target_column": "churn",
  "test_size": 0.2,
  "random_state": 42,
  "feature_columns": ["age", "tenure", "monthly_charges"]
}
```

### Starter Code

```python
import json
from pathlib import Path


def load_model_config(path: str) -> dict:
    # your code here
    pass


# Test 1: valid config
config = load_model_config("model_config.json")
print(config["model_type"])
print(config["feature_columns"])

# Test 2: missing file
try:
    load_model_config("nonexistent.json")
except FileNotFoundError as e:
    print(f"Caught: {e}")
```

### Expected Output

```
random_forest
['age', 'tenure', 'monthly_charges']
Caught: Config file not found: nonexistent.json
```

---

## Exercise 5 — Utility Module (Modules + Pythonic Code)

**Topic:** Creating a module, `if __name__ == "__main__"`

Create a file called `stats_utils.py` with these functions:

```python
def remove_nulls(values: list) -> list:
    """Remove None values from a list."""
    ...

def mean(values: list) -> float | None:
    """Return arithmetic mean, ignoring None values. Return None if empty."""
    ...

def variance(values: list) -> float | None:
    """Return population variance, ignoring None values. Return None if fewer than 2 values."""
    ...

def normalize(values: list) -> list:
    """Min-max normalize to [0, 1]. Return original if all values are equal."""
    ...
```

Include `if __name__ == "__main__":` with test cases.

Then create `main.py` that imports and uses these functions:

```python
from stats_utils import mean, variance, normalize

exam_scores = [88, None, 92, 78, None, 95, 85, None, 70]

print(f"Mean: {mean(exam_scores):.2f}")
print(f"Variance: {variance(exam_scores):.2f}")
print(f"Normalized: {normalize([s for s in exam_scores if s is not None])}")
```

### Expected Output

```
Mean: 84.67
Variance: 62.22
Normalized: [0.727, 1.0, 0.364, 1.0, 0.636, 0.0]
```

(Values approximate — exact output depends on your implementation.)

---

## Exercise 6 — Data Validator Class (OOP + Exception Handling)

**Topic:** Classes, custom exceptions, validation patterns

Build a `RecordValidator` class that validates employee records.

### Requirements

Each record is a dict: `{"name": str, "age": int, "department": str, "salary": float}`

The validator must:

- Check `name` is a non-empty string
- Check `age` is an int in range 18–70
- Check `department` is in an allowed list
- Check `salary` is a positive float

Return a list of error strings (empty list = valid).

### Starter Code

```python
class RecordValidator:
    ALLOWED_DEPARTMENTS = {"Engineering", "Marketing", "Sales", "Finance", "Operations"}

    def __init__(self):
        self._valid_count = 0
        self._invalid_count = 0

    def validate(self, record: dict) -> list[str]:
        # return a list of error strings
        pass

    @property
    def stats(self) -> dict:
        pass


validator = RecordValidator()

test_records = [
    {"name": "Alice", "age": 31, "department": "Engineering", "salary": 95000.0},
    {"name": "", "age": 31, "department": "Engineering", "salary": 95000.0},
    {"name": "Bob", "age": 17, "department": "Marketing", "salary": 55000.0},
    {"name": "Carol", "age": 29, "department": "Unknown", "salary": -1000.0},
]

for rec in test_records:
    errors = validator.validate(rec)
    status = "VALID" if not errors else f"INVALID: {errors}"
    print(f"{rec.get('name', '?')!r:10} — {status}")

print(validator.stats)
```

### Expected Output

```
'Alice'    — VALID
''         — INVALID: ['name must be a non-empty string']
'Bob'      — INVALID: ['age must be an int between 18 and 70']
'Carol'    — INVALID: ["department 'Unknown' not in allowed list", 'salary must be a positive number']
{'total': 4, 'valid': 1, 'invalid': 3, 'pass_rate': 0.25}
```

---

## Exercise 7 — Pythonic Refactoring (Best Practices)

**Topic:** Comprehensions, `enumerate`, `zip`, `any`, `all`

Refactor each snippet to idiomatic Python. Do not change what the code does — only how it does it.

### Snippet A

```python
# Original
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
result = []
i = 0
while i < len(numbers):
    if numbers[i] % 2 == 0:
        result.append(numbers[i] ** 2)
    i += 1
print(result)
```

Expected refactored output: `[4, 16, 36, 64, 100]`

### Snippet B

```python
# Original
products = ["Laptop", "Mouse", "Keyboard"]
prices = [75000, 600, 1200]

print("Products:")
i = 0
for product in products:
    print(str(i + 1) + ". " + product + ": " + str(prices[i]))
    i += 1
```

Expected output:
```
Products:
1. Laptop: 75000
2. Mouse: 600
3. Keyboard: 1200
```

### Snippet C

```python
# Original
scores = [85, 91, 72, 68, 55, 90]

has_failure = False
for s in scores:
    if s < 60:
        has_failure = True
        break

all_pass = True
for s in scores:
    if s < 70:
        all_pass = False
        break

print(has_failure)
print(all_pass)
```

Expected output:
```
True
False
```

---

## Final Challenge — End-to-End Data Pipeline

This exercise combines everything from the session.

### The Task

Write a program that:

1. Reads `raw_employees.csv` (you will create it)
2. Parses and validates each row
3. Skips invalid rows and logs why
4. Saves valid rows to `clean_employees.json`
5. Saves a summary (total rows, valid rows, error log) to `pipeline_summary.json`

### Create the input file

```python
import csv

raw_data = [
    {"name": " alice ", "age": "31", "department": "Engineering", "salary": "95000"},
    {"name": "BOB", "age": "not available", "department": "Marketing", "salary": "72000"},
    {"name": "", "age": "28", "department": "Sales", "salary": "55000"},
    {"name": "carol", "age": "29", "department": "Unknown", "salary": "88000"},
    {"name": "Dave", "age": "34", "department": "Finance", "salary": "invalid"},
    {"name": "Eve", "age": "26", "department": "Engineering", "salary": "78000"},
]

with open("raw_employees.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age", "department", "salary"])
    writer.writeheader()
    writer.writerows(raw_data)
```

### The Pipeline Script

Write `pipeline.py` that:

- Reads the CSV with error handling
- Cleans each record: strip name, title-case, convert types
- Validates: name non-empty, age 18–70, department in allowed set, salary positive
- Saves valid records and a summary

### Expected `clean_employees.json`

```json
[
  {"name": "Alice", "age": 31, "department": "Engineering", "salary": 95000.0},
  {"name": "Eve", "age": 26, "department": "Engineering", "salary": 78000.0}
]
```

### Expected `pipeline_summary.json`

```json
{
  "total_rows": 6,
  "valid_rows": 2,
  "invalid_rows": 4,
  "errors": [
    {"row": 2, "name": "Bob", "reason": "cannot convert age: 'not available'"},
    {"row": 3, "name": "", "reason": "name must be non-empty"},
    {"row": 4, "name": "Carol", "reason": "department 'Unknown' not in allowed list"},
    {"row": 5, "name": "Dave", "reason": "cannot convert salary: 'invalid'"}
  ]
}
```

---

## Self-Check

You are ready for Day 02 if you can:

- [ ] Write a class with `__init__`, instance methods, and `__repr__` from scratch
- [ ] Use `with open()` for CSV reading and JSON writing without looking up the syntax
- [ ] Catch specific exceptions and return a sensible default rather than crashing
- [ ] Import a function from a module you wrote yourself
- [ ] Refactor a `while` loop index into a list comprehension or `enumerate`
- [ ] Explain what `if __name__ == "__main__":` does and why it matters

---

[[05-python-best-practices|← Best Practices]] | [[07-interview-questions|Interview Questions →]]
