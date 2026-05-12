# ⚠️ 04 – Exception Handling

> **Prerequisites:** [[03-modules-and-packages]]  
> **Time to read:** ~25 minutes

---

## 🧠 What is an Exception?

An **exception** is an error that occurs while your program is running. If not handled, it crashes your program with a traceback.

```
Traceback (most recent call last):
  File "main.py", line 5, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero
```

**Exception handling** lets you catch these errors and decide what to do — instead of crashing, your program can log the error, skip bad data, retry, or show a user-friendly message.

---

## 🌳 Python's Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt (Ctrl+C)
└── Exception
    ├── ValueError       (wrong value: int("hello"))
    ├── TypeError        (wrong type: "a" + 1)
    ├── IndexError       (list index out of range)
    ├── KeyError         (dict key not found)
    ├── AttributeError   (object has no attribute)
    ├── NameError        (variable not defined)
    ├── ZeroDivisionError (division by zero)
    ├── FileNotFoundError (file doesn't exist)
    ├── ImportError      (module not found)
    ├── StopIteration
    └── RuntimeError
        └── RecursionError
```

---

## 🛡️ The `try / except` Block

### Basic Syntax

```python
try:
    # Code that might raise an exception
    risky_code()
except ExceptionType:
    # What to do if that exception occurs
    handle_error()
```

### Simple Example

```python
# Without exception handling — CRASHES
result = int("hello")   # ValueError: invalid literal

# With exception handling — GRACEFUL
try:
    result = int("hello")
except ValueError:
    result = 0
    print("Couldn't convert to int, using 0")

print(result)   # 0
```

---

## 🧩 Full `try / except / else / finally`

```python
try:
    # Risky code
    value = int(input("Enter a number: "))
    result = 100 / value

except ValueError:
    # Handles wrong input type
    print("Please enter a valid integer!")
    result = None

except ZeroDivisionError:
    # Handles division by zero
    print("Can't divide by zero!")
    result = None

except (TypeError, AttributeError) as e:
    # Handles multiple exception types
    print(f"Type error: {e}")
    result = None

except Exception as e:
    # Catches any other exception (use sparingly!)
    print(f"Unexpected error: {e}")
    result = None

else:
    # Runs ONLY if no exception occurred
    print(f"Success! Result = {result:.2f}")

finally:
    # ALWAYS runs, exception or not (great for cleanup)
    print("Done processing.")
```

---

## 📋 Common Exceptions & When They Occur

```python
# ValueError — right type, wrong value
int("hello")           # ValueError
float("abc")           # ValueError
math.sqrt(-1)          # ValueError (domain error)

# TypeError — wrong type entirely
"hello" + 5            # TypeError
len(42)                # TypeError

# KeyError — dict key missing
d = {"a": 1}
d["b"]                 # KeyError: 'b'

# IndexError — list index out of bounds
lst = [1, 2, 3]
lst[5]                 # IndexError: list index out of range

# AttributeError — object has no attribute
"hello".nonexistent()  # AttributeError

# FileNotFoundError
open("missing.txt")    # FileNotFoundError

# ZeroDivisionError
10 / 0                 # ZeroDivisionError

# ImportError
import nonexistent     # ModuleNotFoundError (subclass of ImportError)
```

---

## 🚦 Raising Exceptions

You can raise exceptions yourself to signal errors in your code:

```python
def validate_age(age):
    if not isinstance(age, (int, float)):
        raise TypeError(f"Age must be numeric, got {type(age).__name__}")
    if age < 0:
        raise ValueError(f"Age cannot be negative, got {age}")
    if age > 150:
        raise ValueError(f"Age {age} is unrealistically high")
    return int(age)

# Test
try:
    print(validate_age(25))     # 25
    print(validate_age(-5))     # raises ValueError
except ValueError as e:
    print(f"Validation error: {e}")

# Re-raising an exception
def process(data):
    try:
        result = risky_operation(data)
    except ValueError as e:
        print(f"Logging error: {e}")
        raise   # re-raise the same exception upward
```

---

## 🔧 Custom Exceptions

Create your own exception classes for domain-specific errors:

```python
class DataError(Exception):
    """Base exception for data processing errors."""
    pass

class MissingColumnError(DataError):
    """Raised when a required column is missing from the dataset."""
    def __init__(self, column_name, available_columns):
        self.column_name = column_name
        self.available_columns = available_columns
        super().__init__(
            f"Column '{column_name}' not found. "
            f"Available: {available_columns}"
        )

class InvalidValueError(DataError):
    """Raised when a data value is outside acceptable range."""
    def __init__(self, column, value, expected_range):
        self.column = column
        self.value = value
        super().__init__(
            f"Invalid value {value} in column '{column}'. "
            f"Expected: {expected_range}"
        )


# Using custom exceptions
def load_and_validate(data, required_columns):
    available = list(data[0].keys()) if data else []
    for col in required_columns:
        if col not in available:
            raise MissingColumnError(col, available)

    for i, row in enumerate(data):
        age = float(row.get("age", 0))
        if not (0 <= age <= 120):
            raise InvalidValueError("age", age, "0–120")

    return data


# Test
records = [{"name": "Alice", "age": "25"}, {"name": "Bob", "age": "250"}]
try:
    validated = load_and_validate(records, ["name", "age"])
except MissingColumnError as e:
    print(f"Schema error: {e}")
except InvalidValueError as e:
    print(f"Data error: {e}")
```

---

## 🔄 Context Managers (`with` statement)

The `with` statement is Python's built-in exception-safe resource management.

```python
# File automatically closed even if exception occurs
with open("data.txt", "r") as f:
    data = f.read()
    # If an exception occurs here, f is still closed!

# Multiple context managers
with open("input.txt") as fin, open("output.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

### Creating Your Own Context Manager

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label="Operation"):
    """Measure execution time of a block."""
    start = time.time()
    try:
        yield   # code inside the 'with' block runs here
    finally:
        elapsed = time.time() - start
        print(f"{label}: {elapsed:.4f} seconds")

# Usage
with timer("Data Loading"):
    data = list(range(1_000_000))

with timer("Sorting"):
    data.sort()
```

---

## 🧪 Data Science Exception Patterns

### Pattern 1: Safe Data Parsing

```python
def safe_parse_record(raw):
    """Parse a raw string dict into typed values. Return None on failure."""
    try:
        return {
            "name": str(raw["name"]).strip().title(),
            "age": int(raw["age"]),
            "salary": float(raw["salary"]),
        }
    except KeyError as e:
        print(f"Missing field: {e}")
        return None
    except (ValueError, TypeError) as e:
        print(f"Bad value: {e} in {raw}")
        return None

records = [
    {"name": "alice", "age": "30", "salary": "80000"},
    {"name": "bob", "age": "invalid", "salary": "55000"},
    {"name": "charlie", "salary": "70000"},  # missing age
]

cleaned = [safe_parse_record(r) for r in records]
valid = [r for r in cleaned if r is not None]
print(f"Valid: {len(valid)}/{len(records)}")
```

### Pattern 2: Retry Logic

```python
import time

def retry(func, max_attempts=3, delay=1.0, exceptions=(Exception,)):
    """Retry a function on failure with delay."""
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except exceptions as e:
            if attempt == max_attempts:
                raise   # out of retries — re-raise
            print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay)

# Usage
import random
def flaky_api_call():
    if random.random() < 0.7:   # 70% chance of failure
        raise ConnectionError("API timeout")
    return {"data": [1, 2, 3]}

result = retry(flaky_api_call, max_attempts=5, delay=0.5)
```

---

## ✅ Key Takeaways

- `try/except` prevents crashes and lets you handle errors gracefully
- Catch **specific** exceptions (not bare `except:`) — you don't want to accidentally swallow real bugs
- Use `else` for code that runs on success; `finally` for cleanup that always runs
- `raise` lets you signal errors from your own code; always use descriptive messages
- Create **custom exception classes** for domain-specific errors (data validation, ML pipelines)
- `with` (context manager) is the safest way to work with files and resources
- In Data Science: log bad records and skip them rather than crashing on one bad row

---

## 🔗 What's Next?

➡️ [[05-python-best-practices]] — Write clean, Pythonic, professional code