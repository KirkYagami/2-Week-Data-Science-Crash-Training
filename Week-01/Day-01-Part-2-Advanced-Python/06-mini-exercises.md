# 🧪 06 – Mini Exercises: Advanced Python

> **Prerequisites:** [[01-oop-basics]], [[02-file-handling]], [[03-modules-and-packages]], [[04-exception-handling]], [[05-python-best-practices]]  
> **Time to complete:** ~30 minutes

---

## 🎯 Goal

Practice the skills from Day 01 Part 2 by solving small, realistic problems that combine:

- classes and objects
- file reading and writing
- modules from the standard library
- exception handling
- clean, Pythonic code

Work through these in order. Each exercise is intentionally small, but each one mirrors a pattern you will reuse in Data Science projects.

---

## Exercise 1 – Student Class

Create a `Student` class with:

- `name`
- `scores`
- `average()` method
- `grade()` method

### Requirements

- `average()` should return the mean score.
- `grade()` should return:
  - `"A"` for average >= 90
  - `"B"` for average >= 80
  - `"C"` for average >= 70
  - `"D"` for average >= 60
  - `"F"` otherwise

### Starter Code

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        pass

    def grade(self):
        pass


student = Student("Alice", [88, 92, 79])
print(student.average())
print(student.grade())
```

### Expected Output

```text
86.33333333333333
B
```

### Hint

Use `sum()` and `len()`.

---

## Exercise 2 – Safe Number Parser

Write a function called `safe_float(value)` that converts a value to `float`.

### Requirements

- Return the converted number if conversion succeeds.
- Return `None` if conversion fails.
- Handle both `ValueError` and `TypeError`.

### Example

```python
values = ["10.5", "42", "bad", None, "7.25"]

clean_values = []
for value in values:
    number = safe_float(value)
    if number is not None:
        clean_values.append(number)

print(clean_values)
```

### Expected Output

```text
[10.5, 42.0, 7.25]
```

### Hint

```python
try:
    ...
except (ValueError, TypeError):
    ...
```

---

## Exercise 3 – Read and Summarize CSV Data

Create a file called `sales.csv` with this content:

```csv
product,quantity,price
Laptop,2,75000
Mouse,5,600
Keyboard,3,1200
Monitor,2,15000
```

Write a Python script that:

- reads the CSV file
- calculates `total = quantity * price` for each row
- prints each product and its total
- prints the grand total

### Expected Output

```text
Laptop: 150000
Mouse: 3000
Keyboard: 3600
Monitor: 30000
Grand Total: 186600
```

### Starter Code

```python
import csv

grand_total = 0

with open("sales.csv", "r", newline="") as file:
    reader = csv.DictReader(file)

    for row in reader:
        quantity = int(row["quantity"])
        price = float(row["price"])
        total = quantity * price
        grand_total += total
        print(f"{row['product']}: {total:g}")

print(f"Grand Total: {grand_total:g}")
```

### Stretch Goal

Handle bad rows gracefully. If `quantity` or `price` cannot be converted, skip the row and print a warning.

---

## Exercise 4 – JSON Configuration Loader

Create a JSON file called `config.json`:

```json
{
  "dataset_path": "data/customers.csv",
  "model_name": "linear_regression",
  "test_size": 0.2,
  "random_state": 42
}
```

Write a function `load_config(path)` that:

- opens the JSON file
- returns the config as a dictionary
- raises a clear error message if the file does not exist
- raises a clear error message if the JSON is invalid

### Starter Code

```python
import json


def load_config(path):
    try:
        with open(path, "r") as file:
            return json.load(file)
    except FileNotFoundError:
        raise FileNotFoundError(f"Config file not found: {path}")
    except json.JSONDecodeError as error:
        raise ValueError(f"Invalid JSON in config file: {error}")


config = load_config("config.json")
print(config["model_name"])
```

### Expected Output

```text
linear_regression
```

---

## Exercise 5 – Build a Reusable Utility Module

Create a file called `data_utils.py` with these functions:

- `calculate_mean(values)`
- `calculate_min(values)`
- `calculate_max(values)`
- `remove_missing(values)`

Then create another file called `main.py` that imports and uses these functions.

### Example

```python
# data_utils.py

def remove_missing(values):
    return [value for value in values if value is not None]


def calculate_mean(values):
    values = remove_missing(values)
    if not values:
        return None
    return sum(values) / len(values)


def calculate_min(values):
    values = remove_missing(values)
    return min(values) if values else None


def calculate_max(values):
    values = remove_missing(values)
    return max(values) if values else None
```

```python
# main.py

from data_utils import calculate_mean, calculate_min, calculate_max

scores = [80, 90, None, 75, 100]

print(calculate_mean(scores))
print(calculate_min(scores))
print(calculate_max(scores))
```

### Expected Output

```text
86.25
75
100
```

---

## Exercise 6 – Mini Data Validation Class

Create a `DataValidator` class that validates records before they enter a dataset.

### Requirements

Each record is a dictionary:

```python
{"name": "Alice", "age": 25, "salary": 50000}
```

The class should check:

- `name` is present and not empty
- `age` is an integer between 0 and 120
- `salary` is a positive number

### Starter Code

```python
class DataValidator:
    def validate(self, record):
        errors = []

        if not record.get("name"):
            errors.append("name is required")

        age = record.get("age")
        if not isinstance(age, int) or not 0 <= age <= 120:
            errors.append("age must be an integer between 0 and 120")

        salary = record.get("salary")
        if not isinstance(salary, (int, float)) or salary <= 0:
            errors.append("salary must be a positive number")

        return errors


validator = DataValidator()

records = [
    {"name": "Alice", "age": 25, "salary": 50000},
    {"name": "", "age": 130, "salary": -100},
]

for record in records:
    errors = validator.validate(record)
    if errors:
        print("Invalid:", errors)
    else:
        print("Valid:", record)
```

### Expected Output

```text
Valid: {'name': 'Alice', 'age': 25, 'salary': 50000}
Invalid: ['name is required', 'age must be an integer between 0 and 120', 'salary must be a positive number']
```

---

## Exercise 7 – Pythonic Refactoring

Refactor this code to make it more Pythonic:

```python
numbers = [1, 2, 3, 4, 5, 6]

result = []
i = 0
while i < len(numbers):
    if numbers[i] % 2 == 0:
        result.append(numbers[i] * numbers[i])
    i = i + 1

print(result)
```

### Expected Refactor

```python
numbers = [1, 2, 3, 4, 5, 6]
squared_evens = [number**2 for number in numbers if number % 2 == 0]

print(squared_evens)
```

### Expected Output

```text
[4, 16, 36]
```

---

## ✅ Final Challenge – Tiny Data Cleaning Pipeline

Write a program that takes raw employee records, cleans them, validates them, and writes valid records to a JSON file.

### Raw Data

```python
raw_records = [
    {"name": " alice ", "age": "25", "salary": "50000"},
    {"name": "BOB", "age": "not available", "salary": "60000"},
    {"name": "", "age": "31", "salary": "70000"},
    {"name": "charlie", "age": "29", "salary": "invalid"},
]
```

### Requirements

- Strip whitespace from names.
- Convert names to title case.
- Convert `age` to `int`.
- Convert `salary` to `float`.
- Skip invalid records.
- Save valid records to `clean_employees.json`.

### One Possible Solution

```python
import json


def clean_record(record):
    try:
        cleaned = {
            "name": record["name"].strip().title(),
            "age": int(record["age"]),
            "salary": float(record["salary"]),
        }
    except (KeyError, ValueError, TypeError):
        return None

    if not cleaned["name"]:
        return None

    return cleaned


raw_records = [
    {"name": " alice ", "age": "25", "salary": "50000"},
    {"name": "BOB", "age": "not available", "salary": "60000"},
    {"name": "", "age": "31", "salary": "70000"},
    {"name": "charlie", "age": "29", "salary": "invalid"},
]

clean_records = []

for record in raw_records:
    cleaned = clean_record(record)
    if cleaned is not None:
        clean_records.append(cleaned)

with open("clean_employees.json", "w") as file:
    json.dump(clean_records, file, indent=2)

print(clean_records)
```

### Expected Output

```text
[{'name': 'Alice', 'age': 25, 'salary': 50000.0}]
```

---

## ✅ Self-Check

You are ready to move on if you can:

- [ ] create a small class with methods
- [ ] use `with open(...)` for safe file handling
- [ ] read CSV and JSON data
- [ ] handle bad inputs using `try/except`
- [ ] split reusable functions into a module
- [ ] refactor loops into Pythonic code

---

## 🔗 What's Next?

➡️ [[07-interview-questions]] — Practice explaining Advanced Python concepts clearly
