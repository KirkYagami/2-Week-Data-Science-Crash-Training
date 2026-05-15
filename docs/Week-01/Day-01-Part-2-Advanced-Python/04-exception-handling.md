# Exception Handling

A data pipeline that crashes on the first malformed row is not production-ready. Real datasets have encoding errors, missing fields, values that are `"N/A"` where you expected a number, and API calls that time out. Exception handling is how you distinguish between recoverable problems (skip this row and continue) and genuine failures (this should never happen, halt immediately).

**Prerequisites:** [[03-modules-and-packages]]

## Learning Objectives

- Describe what an exception is and trace through how Python propagates one
- Use `try / except / else / finally` correctly and know when each block runs
- Catch specific exceptions rather than using a blank `except:`
- Raise exceptions yourself with informative messages
- Build custom exception classes for domain-specific errors
- Apply the EAFP vs LBYL distinction appropriately

---

## What is an Exception?

An exception is Python's way of signaling that something went wrong at runtime. When an exception is raised and not caught, the program prints a traceback and stops.

```python
# This crashes the program
user_input = "twenty"
age = int(user_input)  # ValueError: invalid literal for int() with base 10: 'twenty'
```

The traceback tells you exactly what went wrong and where:

```
Traceback (most recent call last):
  File "main.py", line 2, in <module>
    age = int(user_input)
ValueError: invalid literal for int() with base 10: 'twenty'
```

Read tracebacks from the bottom up. The last line tells you the exception type and message. The lines above show the call stack — where in your code the failure originated.

---

## Python's Exception Hierarchy

Exceptions form a hierarchy. Catching a parent class catches all its children.

```
BaseException
├── SystemExit              — raised by sys.exit()
├── KeyboardInterrupt       — raised by Ctrl+C
└── Exception               — the one you normally catch
    ├── ValueError          — right type, wrong value: int("hello")
    ├── TypeError           — wrong type: "a" + 1
    ├── KeyError            — dict key missing: d["nonexistent"]
    ├── IndexError          — list position out of range: lst[99]
    ├── AttributeError      — object has no such attribute
    ├── NameError           — variable not defined
    ├── ZeroDivisionError   — division by zero
    ├── FileNotFoundError   — file or directory missing
    ├── PermissionError     — insufficient permissions to access file
    ├── ImportError         — module not found (ModuleNotFoundError is a subclass)
    ├── OSError             — parent of many filesystem errors
    └── RuntimeError        — generic runtime error
        └── RecursionError  — maximum recursion depth exceeded
```

> [!info]
> `SystemExit` and `KeyboardInterrupt` do not inherit from `Exception`. This is why `except Exception` does not accidentally swallow a Ctrl+C signal — they are separate branches.

---

## The `try / except` Block

```python
# The basic pattern
try:
    # Code that might raise an exception
    result = int("not a number")
except ValueError:
    # Runs only if a ValueError was raised in the try block
    result = 0
    print("Could not convert — defaulting to 0")

print(result)  # Output: 0
```

### Catching Multiple Exceptions

```python
def parse_record(raw: dict) -> dict:
    """Parse a raw string dict into typed values."""
    try:
        return {
            "name": str(raw["name"]).strip().title(),
            "age": int(raw["age"]),
            "salary": float(raw["salary"]),
        }
    except KeyError as e:
        print(f"Missing field: {e}")
        return None
    except ValueError as e:
        print(f"Cannot convert value: {e} — record: {raw}")
        return None
    except TypeError as e:
        print(f"Unexpected type: {e}")
        return None


test_records = [
    {"name": "alice", "age": "31", "salary": "72000"},
    {"name": "bob", "age": "not available", "salary": "55000"},
    {"name": "carol", "salary": "88000"},  # missing age
]

for rec in test_records:
    parsed = parse_record(rec)
    if parsed:
        print(parsed)

# Output:
# {'name': 'Alice', 'age': 31, 'salary': 72000.0}
# Cannot convert value: invalid literal for int() with base 10: 'not available' — record: {...}
# Missing field: 'age'
```

### Catching Multiple Types in One Clause

```python
try:
    value = float(raw_value)
except (ValueError, TypeError):
    # Both are handled the same way
    value = None
```

---

## Full `try / except / else / finally`

```python
import json
from pathlib import Path

def load_config(config_path: str) -> dict | None:
    """Load a JSON config file with clear error handling."""
    path = Path(config_path)

    try:
        with open(path, "r", encoding="utf-8") as f:
            config = json.load(f)

    except FileNotFoundError:
        print(f"Config not found: {path}")
        return None

    except json.JSONDecodeError as e:
        print(f"Invalid JSON in {path}: {e}")
        return None

    else:
        # Runs ONLY if no exception occurred in try
        print(f"Config loaded: {len(config)} keys")
        return config

    finally:
        # ALWAYS runs — exception or not, return or not
        # Use for cleanup: closing connections, logging, releasing locks
        print(f"Finished processing {path}")


result = load_config("config.json")
# If file exists and is valid:
#   Output: Config loaded: 4 keys
#           Finished processing config.json

# If file is missing:
#   Output: Config not found: config.json
#           Finished processing config.json
```

When does each block run?

| Block | Runs when |
|-------|-----------|
| `try` | Always — this is where the risky code lives |
| `except` | Only when the matching exception is raised |
| `else` | Only when NO exception was raised in `try` |
| `finally` | Always — even if there is a `return` in `try` or `except` |

> [!tip]
> Put cleanup logic in `finally`. Database connections, file handles, and network sockets should be closed there — they close whether the operation succeeded or failed.

---

## Common Exceptions and When They Occur

```python
# ValueError — correct type, wrong value
int("hello")           # ValueError: invalid literal for int()
math.sqrt(-1)          # ValueError: math domain error
[].index("missing")    # ValueError: 'missing' is not in list


# TypeError — wrong type for the operation
"hello" + 5            # TypeError: can only concatenate str (not "int") to str
len(42)                # TypeError: object of type 'int' has no len()
None + 1               # TypeError: unsupported operand type(s)


# KeyError — dict key does not exist
d = {"name": "Alice"}
d["age"]               # KeyError: 'age'
# Use .get() instead: d.get("age") returns None
# Or: d.get("age", 0) returns 0 as default


# IndexError — list position out of bounds
lst = [1, 2, 3]
lst[5]                 # IndexError: list index out of range


# AttributeError — object does not have that attribute
"hello".nonexistent_method()   # AttributeError
None.strip()                   # AttributeError: 'NoneType' has no attribute 'strip'


# FileNotFoundError
open("missing.txt")    # FileNotFoundError: [Errno 2] No such file or directory


# ZeroDivisionError
10 / 0                 # ZeroDivisionError: division by zero
# Safe pattern: check before dividing
result = 10 / denominator if denominator != 0 else 0


# ImportError / ModuleNotFoundError
import nonexistent_module   # ModuleNotFoundError: No module named 'nonexistent_module'
```

---

## Raising Exceptions

Raise exceptions yourself when your code cannot proceed safely. Always include a message that tells the caller what went wrong and what was expected.

```python
def calculate_churn_rate(churned_count: int, total_customers: int) -> float:
    """Return churn rate as a value between 0 and 1."""
    if not isinstance(churned_count, int):
        raise TypeError(
            f"churned_count must be int, got {type(churned_count).__name__}"
        )
    if not isinstance(total_customers, int):
        raise TypeError(
            f"total_customers must be int, got {type(total_customers).__name__}"
        )
    if churned_count < 0:
        raise ValueError(f"churned_count cannot be negative, got {churned_count}")
    if total_customers <= 0:
        raise ValueError(
            f"total_customers must be positive, got {total_customers}"
        )
    if churned_count > total_customers:
        raise ValueError(
            f"churned_count ({churned_count}) cannot exceed "
            f"total_customers ({total_customers})"
        )
    return churned_count / total_customers


print(calculate_churn_rate(150, 1000))  # Output: 0.15

try:
    print(calculate_churn_rate(-5, 1000))
except ValueError as e:
    print(f"Error: {e}")  # Output: Error: churned_count cannot be negative, got -5
```

### Re-raising Exceptions

Sometimes you want to log an error and then let it propagate up to the caller:

```python
import logging

logger = logging.getLogger(__name__)


def load_training_data(filepath: str) -> list:
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)
        return data
    except FileNotFoundError:
        logger.error(f"Training data not found: {filepath}")
        raise  # re-raise the same exception — caller handles it
    except json.JSONDecodeError as e:
        logger.error(f"Corrupt training data at {filepath}: {e}")
        raise ValueError(f"Cannot parse training data: {filepath}") from e
        # 'from e' chains the exceptions — the original is preserved in traceback
```

---

## Custom Exception Classes

When your codebase grows, generic exceptions like `ValueError` do not tell callers much. Custom exceptions let you name the exact failure mode.

```python
class PipelineError(Exception):
    """Base exception for all data pipeline errors."""
    pass


class MissingColumnError(PipelineError):
    """A required column is absent from the dataset."""

    def __init__(self, column: str, available: list):
        self.column = column
        self.available = available
        super().__init__(
            f"Required column '{column}' not found. "
            f"Available columns: {available}"
        )


class DataRangeError(PipelineError):
    """A value is outside the acceptable range."""

    def __init__(self, column: str, value, expected: str):
        self.column = column
        self.value = value
        self.expected = expected
        super().__init__(
            f"Column '{column}': value {value!r} is out of range. "
            f"Expected: {expected}"
        )


class EmptyDatasetError(PipelineError):
    """The dataset has no records."""
    pass


def validate_dataset(records: list, required_columns: list) -> list:
    """Validate records and raise domain-specific exceptions on failure."""
    if not records:
        raise EmptyDatasetError("Dataset has no records — cannot proceed")

    available = list(records[0].keys())
    for col in required_columns:
        if col not in available:
            raise MissingColumnError(col, available)

    for i, record in enumerate(records):
        age = record.get("age")
        if age is not None and not (0 <= float(age) <= 120):
            raise DataRangeError("age", age, "0 to 120")

    return records


# Usage
sample = [
    {"name": "Alice", "age": 31, "salary": 72000},
    {"name": "Bob", "age": 28, "salary": 55000},
]

try:
    validated = validate_dataset(sample, required_columns=["name", "age", "salary"])
    print(f"Validated {len(validated)} records")  # Output: Validated 2 records
except MissingColumnError as e:
    print(f"Schema error: {e}")
except DataRangeError as e:
    print(f"Bad data: {e}")
except EmptyDatasetError as e:
    print(f"Empty input: {e}")
```

> [!tip]
> Always define a base exception class for your project (e.g., `PipelineError`). This lets callers catch all your errors with one `except PipelineError` while still allowing specific handling of `MissingColumnError` vs `DataRangeError` when needed.

---

## EAFP vs LBYL

Python culture has two philosophies for dealing with potential errors:

**LBYL — Look Before You Leap**: Check conditions before acting.

```python
# LBYL style
import os
if os.path.exists("data.csv") and os.path.isfile("data.csv"):
    with open("data.csv") as f:
        data = f.read()
```

**EAFP — Easier to Ask Forgiveness than Permission**: Try the action, catch the failure.

```python
# EAFP style — preferred in Python
try:
    with open("data.csv") as f:
        data = f.read()
except FileNotFoundError:
    data = None
```

> [!info]
> Python idiom favors EAFP because it avoids race conditions (the file could be deleted between your `os.path.exists()` check and the `open()` call), and it is usually shorter. LBYL makes sense for simple conditions like `if denominator != 0`.

---

## Practical Data Science Pattern — Safe Row Processing

```python
import csv
from pathlib import Path


def process_sales_file(filepath: str) -> dict:
    """
    Process a sales CSV and return summary statistics.
    Skips malformed rows without crashing.
    """
    valid_rows = []
    error_rows = []
    path = Path(filepath)

    if not path.exists():
        raise FileNotFoundError(f"Sales file not found: {filepath}")

    with open(path, "r", newline="", encoding="utf-8") as f:
        for row_num, row in enumerate(csv.DictReader(f), start=2):
            try:
                processed = {
                    "product": row["product"].strip(),
                    "units_sold": int(row["units_sold"]),
                    "unit_price": float(row["unit_price"]),
                    "revenue": int(row["units_sold"]) * float(row["unit_price"]),
                }
                if processed["units_sold"] < 0:
                    raise ValueError(f"Negative units_sold: {processed['units_sold']}")
                valid_rows.append(processed)
            except KeyError as e:
                error_rows.append({"row": row_num, "reason": f"Missing column: {e}"})
            except (ValueError, TypeError) as e:
                error_rows.append({"row": row_num, "reason": str(e)})

    total_revenue = sum(r["revenue"] for r in valid_rows)

    return {
        "valid_count": len(valid_rows),
        "error_count": len(error_rows),
        "total_revenue": total_revenue,
        "errors": error_rows,
        "records": valid_rows,
    }


# If you had a sales.csv file:
# results = process_sales_file("data/sales.csv")
# print(f"Processed {results['valid_count']} rows, {results['error_count']} errors")
# print(f"Total revenue: {results['total_revenue']:,.2f}")
```

---

## Logging vs Printing Errors

In production code, use `logging` instead of `print()` for error messages. Logging gives you levels (DEBUG, INFO, WARNING, ERROR, CRITICAL), timestamps, and the ability to route messages to files.

```python
import logging

# Configure once at the top of your script or in main()
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s — %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logger = logging.getLogger(__name__)


def load_model_config(path: str) -> dict:
    import json
    try:
        with open(path, "r") as f:
            config = json.load(f)
        logger.info(f"Loaded config from {path}")
        return config
    except FileNotFoundError:
        logger.error(f"Config file missing: {path}")
        raise
    except json.JSONDecodeError as e:
        logger.error(f"Invalid JSON in {path}: {e}")
        raise


# Output when working correctly:
# 2025-01-15 14:30:22 [INFO] __main__ — Loaded config from config.json

# Output when file is missing:
# 2025-01-15 14:30:22 [ERROR] __main__ — Config file missing: config.json
```

> [!warning]
> Never use `print()` for error reporting in code that will run in production or in a team. `print()` goes to stdout, gets mixed with other output, and is invisible in log monitoring systems. Use the `logging` module from day one.

---

## Key Takeaways

> [!success]
> - `try / except` prevents crashes and lets you decide what to do about specific failures
> - Catch **specific** exception types — a bare `except:` swallows system signals and hides bugs
> - `else` runs when the `try` block succeeds; `finally` runs no matter what
> - `raise` with a descriptive message makes debugging far easier for the next person (often you)
> - Custom exception classes make your error handling self-documenting
> - Prefer EAFP over LBYL in Python — try the action, catch the failure
> - Use `logging` instead of `print()` for error reporting

---

[[03-modules-and-packages|← Modules & Packages]] | [[05-python-best-practices|Best Practices →]]
