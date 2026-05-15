# Variables and Data Types

Every program you write manipulates data. Before you can manipulate it, you need to know how Python stores it, what kinds of values exist, and how Python decides what operations are valid. Get this wrong and you will spend hours debugging type errors and silent precision losses that corrupt your analysis results.

---

## Learning Objectives

By the end of this note, you will be able to:

- Declare variables using correct naming conventions
- Distinguish between `int`, `float`, `str`, `bool`, and `NoneType`
- Explain why `0.1 + 0.2 != 0.3` and handle floating point precision correctly
- Use f-strings for readable, formatted output
- Understand truthy and falsy values and use them in conditionals
- Convert between types safely and catch conversion failures
- Explain the difference between `type()` and `isinstance()`
- Use multiple assignment and tuple unpacking

---

## What is a Variable?

A variable is a name bound to a value in memory. When you write `age = 25`, Python creates the integer object `25` in memory and binds the name `age` to it. If you then write `age = 26`, Python creates a new integer object `26` and rebinds `age` to point at it. The old `25` object is garbage-collected if nothing else references it.

```python
age = 25
print(id(age))    # memory address, e.g. 140234567

age = 26
print(id(age))    # different address — a new object!
```

This matters because Python variables are references, not containers. When you write `b = a`, both names point to the same object — they do not copy it.

```python
# With immutable types (int, str), this doesn't cause problems
a = 10
b = a
b = 20
print(a)    # Output: 10 — a is unchanged

# With mutable types (list), it does
scores = [85, 92, 78]
backup = scores             # backup points to the SAME list
backup.append(100)
print(scores)               # Output: [85, 92, 78, 100] — surprise!
```

> [!warning]
> This is one of the most common bugs in data science code. When you "copy" a list or dict by assignment, you get a reference to the same object. Use `scores.copy()` or `scores[:]` for a shallow copy, or `copy.deepcopy(scores)` for nested structures.

---

## Variable Naming Rules

### Rules you must follow

| Rule | Wrong | Right |
|------|-------|-------|
| Start with a letter or underscore | `1count = 0` | `count = 0` |
| Only letters, digits, underscores | `user-name = "x"` | `user_name = "x"` |
| Case-sensitive | `Age` and `age` are different | Pick one and stick to it |
| Cannot be a reserved keyword | `if = 5` | `is_valid = 5` |

### Python reserved keywords

These names are claimed by the language itself and cannot be used as variable names:

```
False    None     True     and      as       assert   async
await    break    class    continue def      del      elif
else     except   finally  for      from     global   if
import   in       is       lambda   nonlocal not      or
pass     raise    return   try      while    with     yield
```

### Conventions you should follow

```python
# snake_case for variables and functions — Python standard
customer_age = 34
total_revenue = 285000.50

# UPPER_CASE for constants
MAX_RETRIES = 3
DEFAULT_TIMEOUT_SECONDS = 30

# Descriptive names — make the intent obvious
# Poor names
x = 34
d = "2024-01-15"
lst = [85, 92, 78, 61]

# Good names
customer_age = 34
transaction_date = "2024-01-15"
exam_scores = [85, 92, 78, 61]
```

> [!tip]
> In data science, you will often work with DataFrames where column names become variable names. Choose column names that read as clear descriptions: `purchase_amount` not `amt`, `is_churned` not `churn`.

---

## Python's Core Data Types

```
Python Types
│
├── Numeric
│   ├── int      integers: -5, 0, 42, 1_000_000
│   ├── float    decimals: 3.14, -0.001, 2.0, 6.022e23
│   └── complex  rarely used in DS: 3+4j
│
├── Text
│   └── str      "hello", 'world', """multiline"""
│
├── Boolean
│   └── bool     True, False
│
├── None
│   └── NoneType represents absence of a value
│
└── Collections  (covered in depth in note 05)
    ├── list     [1, 2, 3]
    ├── tuple    (1, 2, 3)
    ├── dict     {"key": "value"}
    └── set      {1, 2, 3}
```

---

## Integers (`int`)

Integers are whole numbers — no decimal point.

```python
user_count = 42
temperature_celsius = -10
world_population = 8_000_000_000    # underscores improve readability (Python 3.6+)

print(type(user_count))             # Output: <class 'int'>
```

### Arithmetic operations

```python
items_sold = 17
price_per_item = 5

total = items_sold * price_per_item     # Output: 85
remainder = items_sold % 3              # Output: 2  — useful for grouping
batches = items_sold // 3              # Output: 5  — floor division
print(2 ** 10)                         # Output: 1024 — exponentiation
```

> [!info]
> Python integers have unlimited precision. Unlike most languages, Python will happily compute `2 ** 1000` — a 302-digit number — without overflow. This matters when working with combinatorics or cryptography, but for data science you will rarely hit practical limits.

### Division always produces a float

```python
result = 10 / 2
print(result)           # Output: 5.0 — not 5!
print(type(result))     # Output: <class 'float'>
```

This trips up developers coming from Python 2, where `10 / 2` was `5` (integer division). In Python 3, use `//` for integer division.

---

## Floats (`float`)

Floats represent decimal numbers. Internally they use IEEE 754 double precision — 64 bits.

```python
item_price = 19.99
pi = 3.14159265358979
avogadro = 6.022e23        # scientific notation: 6.022 × 10^23
planck = 6.626e-34

print(type(item_price))    # Output: <class 'float'>
```

### The floating point precision trap

This surprises almost every programmer at some point:

```python
print(0.1 + 0.2)
# Output: 0.30000000000000004
# Expected: 0.3
```

The cause: `0.1` cannot be represented exactly in binary floating point, just as `1/3` cannot be represented exactly in decimal. The error accumulates during addition.

```python
# Option 1: Round for display
result = 0.1 + 0.2
print(round(result, 2))                         # Output: 0.3

# Option 2: Use the decimal module for financial calculations
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))          # Output: 0.3  — exact

# Option 3: Compare with tolerance (standard in scientific code)
import math
print(math.isclose(0.1 + 0.2, 0.3))            # Output: True
```

> [!warning]
> Never use `==` to compare two floats. `0.1 + 0.2 == 0.3` is `False`. Use `math.isclose()` or check `abs(a - b) < 1e-9`. In production financial code I have seen this cause incorrect account balances accumulate over millions of transactions.

---

## Strings (`str`)

Strings are sequences of Unicode characters. They are immutable — you cannot change a character in place, you create a new string.

```python
# Three quoting styles — same result
first_name = "Alice"
last_name = 'Sharma'
biography = """She is a data scientist
who specializes in NLP
and time series analysis."""
```

### Indexing and slicing

```python
language = "Python"
#           012345   (positive indices)
#          -6-5-4-3-2-1  (negative indices)

print(language[0])      # Output: P
print(language[-1])     # Output: n  — last character
print(language[0:3])    # Output: Pyt  — stop index is exclusive
print(language[2:])     # Output: thon
print(language[::-1])   # Output: nohtyP  — reversed
```

### Essential string methods

```python
raw_input = "  Alice Sharma, Data Scientist  "

# Cleaning
print(raw_input.strip())                    # Output: Alice Sharma, Data Scientist
print(raw_input.lower())                    # Output:   alice sharma, data scientist
print(raw_input.upper())                    # Output:   ALICE SHARMA, DATA SCIENTIST

# Searching
print("Sharma" in raw_input)               # Output: True
print(raw_input.find("Data"))              # Output: 16  (index of match, -1 if not found)
print(raw_input.count("a"))                # Output: 4

# Replacing and splitting
clean = raw_input.strip()
print(clean.replace("Data Scientist", "ML Engineer"))
# Output: Alice Sharma, ML Engineer

parts = clean.split(", ")
print(parts)                               # Output: ['Alice Sharma', 'Data Scientist']

# Joining (the inverse of split)
fields = ["Alice", "30", "Mumbai"]
csv_row = ",".join(fields)
print(csv_row)                             # Output: Alice,30,Mumbai

# Validation
print("42".isdigit())                      # Output: True
print("hello".isalpha())                   # Output: True
print("hello42".isalnum())                 # Output: True
```

### String formatting — use f-strings

Python has three formatting styles. Use f-strings for all new code.

```python
customer_name = "Priya"
order_total = 1234.5678
item_count = 3

# f-strings (Python 3.6+) — recommended
print(f"Customer: {customer_name}")                   # Output: Customer: Priya
print(f"Total: {order_total:.2f}")                    # Output: Total: 1234.57
print(f"Items: {item_count:04d}")                     # Output: Items: 0003
print(f"Revenue: {order_total:,.2f}")                 # Output: Revenue: 1,234.57

# Alignment — useful for tabular output
print(f"{'Name':<15} {'Score':>8}")
print(f"{'Alice':<15} {85:>8}")
print(f"{'Bob':<15} {92:>8}")
# Output:
# Name                Score
# Alice                  85
# Bob                    92

# Expressions inside f-strings
radius = 5
print(f"Area: {3.14159 * radius ** 2:.1f}")           # Output: Area: 78.5

# Older styles — you will encounter these in existing code
print("Customer: %s, Total: %.2f" % (customer_name, order_total))
print("Customer: {}, Total: {:.2f}".format(customer_name, order_total))
```

> [!info]
> f-strings are not just more readable — they are also faster than `.format()` and `%` formatting at runtime. They also catch missing variable names at parse time rather than runtime, which surfaces bugs earlier.

---

## Booleans (`bool`)

Booleans have exactly two values: `True` and `False`. They are a subtype of `int` — `True` equals `1` and `False` equals `0`.

```python
is_active = True
has_missing_data = False

print(type(is_active))          # Output: <class 'bool'>
print(True + True)              # Output: 2  — Booleans are ints
print(True * 100)               # Output: 100
print(sum([True, False, True, True]))  # Output: 3 — count of True values
```

### Boolean operators

```python
# and — both must be True
print(True and True)             # Output: True
print(True and False)            # Output: False

# or — at least one must be True
print(True or False)             # Output: True
print(False or False)            # Output: False

# not — inverts
print(not True)                  # Output: False
print(not False)                 # Output: True
```

### Comparison operators return booleans

```python
revenue = 85000
target = 100000

print(revenue == target)         # Output: False
print(revenue != target)         # Output: True
print(revenue < target)          # Output: True
print(revenue >= target)         # Output: False

# Chained comparisons — very Pythonic
age = 34
print(18 <= age <= 65)           # Output: True — is age in working-age range?

# Combining comparisons
score = 87
attendance = 80
passed = score >= 60 and attendance >= 75
print(passed)                    # Output: True
```

### Truthy and falsy values

Python evaluates any value as `True` or `False` in a boolean context. Understanding this makes your conditionals cleaner.

```python
# Falsy — these all evaluate to False in an if/while condition
bool(0)         # False
bool(0.0)       # False
bool("")        # False — empty string
bool([])        # False — empty list
bool({})        # False — empty dict
bool(set())     # False — empty set
bool(None)      # False

# Everything else is truthy
bool(1)         # True
bool(-0.1)      # True — any nonzero number
bool("x")       # True — any nonempty string
bool([0])       # True — a list with one element (even if that element is falsy!)
```

```python
# Practical use — check for empty results before processing
def get_top_customers(threshold=1000):
    # Imagine this queries a database
    results = []   # empty for now
    return results

customers = get_top_customers()

if customers:
    print(f"Processing {len(customers)} customers")
else:
    print("No customers above threshold — check your filter")
# Output: No customers above threshold — check your filter
```

> [!tip]
> In data pipelines, checking `if df.empty:` or `if not results:` before processing prevents downstream errors on empty datasets. Make it a habit.

---

## None (`NoneType`)

`None` represents the absence of a value. It is Python's equivalent of "nothing here," "missing," or "not yet set."

```python
prediction_result = None    # model has not been run yet

print(prediction_result)     # Output: None
print(type(prediction_result))  # Output: <class 'NoneType'>
```

### How to check for None

```python
# Correct — use `is`, not `==`
if prediction_result is None:
    print("Model has not run yet")

if prediction_result is not None:
    print(f"Prediction: {prediction_result}")

# This also works but is less Pythonic
if prediction_result == None:
    print("Model has not run yet")
```

> [!warning]
> Use `is None` rather than `== None`. The `is` operator checks object identity, which is guaranteed to work correctly for `None`. The `==` operator can be overridden by custom classes to return unexpected results.

> [!info]
> In Pandas, missing values are represented as `NaN` (Not a Number) — a float value from the NumPy library. When you load data from CSV files, `None` in your Python code often becomes `NaN` in a DataFrame. They are related but not identical — `None` is a Python singleton, `NaN` is a special float value.

---

## Type Conversion

Python provides built-in functions to convert between types.

```python
# String to number — common when reading CSV data
row_value = "42"
count = int(row_value)         # Output: 42
print(type(count))             # Output: <class 'int'>

price_str = "19.99"
price = float(price_str)       # Output: 19.99

# Number to string
record_id = 1001
id_str = str(record_id)        # Output: "1001"
print("Record: " + id_str)     # Output: Record: 1001  — concatenation needs strings

# int() truncates, it does not round
print(int(3.9))                # Output: 3  — not 4!
print(int(-3.9))               # Output: -3  — not -4!

# Boolean conversions
print(int(True))               # Output: 1
print(int(False))              # Output: 0
print(bool(0))                 # Output: False
print(bool(""))                # Output: False
print(bool("False"))           # Output: True  — nonempty string!
```

### Handling conversion failures

```python
# These raise ValueError
# int("hello")      # ValueError: invalid literal for int()
# float("N/A")      # ValueError: could not convert string to float

# Safe conversion pattern — use this in data cleaning code
def to_float(value, default=None):
    """Convert a value to float, returning default if conversion fails."""
    try:
        return float(value)
    except (ValueError, TypeError):
        return default

# Cleaning messy data from a spreadsheet
raw_values = ["19.99", "N/A", "25.00", "", "invalid", "30.50"]
clean_prices = [to_float(v, default=0.0) for v in raw_values]
print(clean_prices)
# Output: [19.99, 0.0, 25.0, 0.0, 0.0, 30.5]
```

> [!warning]
> `int("3.14")` raises a `ValueError` — Python will not convert a float-looking string directly to int. You need `int(float("3.14"))`. This comes up frequently when parsing CSV data where numbers have decimal points.

---

## Checking Types

```python
age = 34
score = 87.5
name = "Alice"
is_active = True

# type() — returns the exact type
print(type(age))              # Output: <class 'int'>
print(type(score))            # Output: <class 'float'>
print(type(name))             # Output: <class 'str'>

# isinstance() — preferred, handles inheritance
print(isinstance(age, int))         # Output: True
print(isinstance(score, float))     # Output: True
print(isinstance(name, str))        # Output: True

# isinstance checks inheritance — bool is a subclass of int
print(isinstance(is_active, bool))  # Output: True
print(isinstance(is_active, int))   # Output: True  — bool IS an int

# Use isinstance when you want to accept multiple types
def process_value(value):
    if isinstance(value, (int, float)):
        return value * 2
    elif isinstance(value, str):
        return value.upper()
    else:
        raise TypeError(f"Cannot process type: {type(value).__name__}")

print(process_value(21))          # Output: 42
print(process_value(3.14))        # Output: 6.28
print(process_value("hello"))     # Output: HELLO
```

---

## Dynamic Typing

Python is dynamically typed — the type is attached to the value, not the variable name. The same name can point to different types at different times.

```python
result = 42
print(type(result))     # Output: <class 'int'>

result = "forty-two"
print(type(result))     # Output: <class 'str'>

result = [4, 2]
print(type(result))     # Output: <class 'list'>
```

This is convenient during exploration but can cause subtle bugs in production code. Python 3.5+ added optional type hints to address this:

```python
def calculate_discount(price: float, rate: float) -> float:
    """Return the discounted price."""
    return price * (1 - rate)

# Type hints are not enforced at runtime — they are documentation and tooling hints
result = calculate_discount(100.0, 0.2)
print(result)    # Output: 80.0
```

> [!tip]
> Add type hints to any function you write in a production codebase. They do not change runtime behavior, but they let editors warn you about type mismatches before you run the code.

---

## Multiple Assignment and Unpacking

```python
# Assign multiple variables in one line
x, y, z = 1, 2, 3
print(x, y, z)               # Output: 1 2 3

# Assign the same value to multiple variables
minimum = maximum = 0
print(minimum, maximum)      # Output: 0 0

# Swap without a temporary variable
latitude = 40.7128
longitude = -74.0060
latitude, longitude = longitude, latitude    # swap
print(latitude, longitude)   # Output: -74.006 40.7128

# Unpack from a list or tuple
coordinates = (48.8566, 2.3522)    # Paris
lat, lon = coordinates
print(f"Paris — lat: {lat}, lon: {lon}")
# Output: Paris — lat: 48.8566, lon: 2.3522

# Extended unpacking with *
first, *rest = [10, 20, 30, 40, 50]
print(first)    # Output: 10
print(rest)     # Output: [20, 30, 40, 50]

head, *middle, tail = [10, 20, 30, 40, 50]
print(head, middle, tail)    # Output: 10 [20, 30, 40] 50
```

---

## Data Types in Data Science — Reference

| Python type | Pandas dtype | Typical use |
|-------------|-------------|-------------|
| `int` | `int64` | Counts, IDs, ages, years |
| `float` | `float64` | Prices, measurements, probabilities, scores |
| `str` | `object` | Names, categories, raw text |
| `bool` | `bool` | Flags, binary features, filter masks |
| `None` | `NaN` | Missing values before and after loading |

---

## Key Takeaways

> [!success]
> - Variables are names bound to objects in memory — assignment does not copy mutable objects
> - Python has four core scalar types: `int`, `float`, `str`, and `bool`
> - `None` is the explicit representation of "no value" — check for it with `is None`
> - Float arithmetic has precision limits — never compare floats with `==`
> - Use f-strings for all string formatting — they are readable, fast, and catch errors early
> - `isinstance()` is better than `type()` because it handles inheritance
> - Python is dynamically typed — add type hints to production functions as documentation

---

[[01-python-introduction]] | [[03-control-flow]]
