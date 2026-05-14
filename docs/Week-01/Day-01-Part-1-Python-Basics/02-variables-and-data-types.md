# 📦 02 – Variables and Data Types

> **Prerequisites:** [[01-python-introduction]]  
> **Time to read:** ~30 minutes

---

## 🧠 What is a Variable?

A **variable** is a named container that stores a value in your computer's memory.

Think of it like a **labelled box**:
- The **label** is the variable name
- The **contents** are the value
- You can change the contents at any time

```python
# Creating variables
age = 25           # label: "age"  |  contents: 25
name = "Alice"     # label: "name" |  contents: "Alice"
height = 5.7       # label: "height" | contents: 5.7
```

### How Variables Work in Memory

```
Variable Name    Memory Address    Value
    age     →    [0x7f3a...]   →   25
    name    →    [0x7f3b...]   →   "Alice"
    height  →    [0x7f3c...]   →   5.7
```

When you type `age`, Python looks up that label and retrieves `25`.

---

## 📝 Variable Naming Rules

### Rules (you MUST follow these)

| Rule | Example of violation | Correct |
|------|---------------------|---------|
| Start with letter or `_` | `1name = "Alice"` ❌ | `name1 = "Alice"` ✅ |
| Only letters, digits, `_` | `my-name = "Alice"` ❌ | `my_name = "Alice"` ✅ |
| Case-sensitive | `Age` ≠ `age` ≠ `AGE` | Pick one and stick to it |
| Can't use reserved words | `if = 5` ❌ | `is_valid = 5` ✅ |

### Python Reserved Words (don't use as variable names)

```
False    None     True     and      as       assert   async
await    break    class    continue def      del      elif
else     except   finally  for      from     global   if
import   in       is       lambda   nonlocal not      or
pass     raise    return   try      while    with     yield
```

### Conventions (you SHOULD follow these)

```python
# snake_case for variables and functions (Python standard)
user_age = 25
total_sales = 1500.75

# UPPER_CASE for constants
MAX_RETRY_COUNT = 3
PI = 3.14159

# Descriptive names (not single letters, except in math/loops)
# BAD
x = 25
d = "Alice"

# GOOD
age = 25
student_name = "Alice"
```

---

## 🔢 Python's Core Data Types

Python has several built-in data types. For Data Science, these are the most important:

```
Python Data Types
│
├── Numeric
│   ├── int      (integers: -5, 0, 42, 1000000)
│   ├── float    (decimals: 3.14, -0.001, 2.0)
│   └── complex  (rarely used in DS: 3+4j)
│
├── Text
│   └── str      ("hello", 'world', """multiline""")
│
├── Boolean
│   └── bool     (True, False)
│
├── None
│   └── NoneType (represents "nothing" / missing)
│
└── Collections (covered in detail later)
    ├── list     ([1, 2, 3])
    ├── tuple    ((1, 2, 3))
    ├── dict     ({"key": "value"})
    └── set      ({1, 2, 3})
```

---

## 🔢 Integers (`int`)

Integers are **whole numbers** — no decimal point.

```python
# Integer examples
count = 42
temperature = -10
population = 8_000_000_000   # underscores for readability (Python 3.6+)
zero = 0

# Check the type
print(type(count))           # <class 'int'>
print(type(temperature))     # <class 'int'>
```

### Integer Operations

```python
a = 17
b = 5

print(a + b)    # 22   — Addition
print(a - b)    # 12   — Subtraction
print(a * b)    # 85   — Multiplication
print(a / b)    # 3.4  — Division (always returns float!)
print(a // b)   # 3    — Floor division (discard remainder)
print(a % b)    # 2    — Modulo (remainder)
print(a ** b)   # 1419857 — Exponentiation (17^5)
```

> **Data Science tip:** Integer division `//` and modulo `%` are useful for splitting datasets, pagination, and working with time.

### Big integers — Python handles them perfectly

```python
# Python integers have unlimited precision (unlike most languages)
huge = 2 ** 1000
print(huge)  # prints a 302-digit number!
```

---

## 🔣 Floats (`float`)

Floats represent **decimal numbers**.

```python
# Float examples
price = 19.99
pi = 3.14159265358979
temperature = -273.15
scientific = 6.022e23        # 6.022 × 10^23 (scientific notation)
small = 1.5e-10              # 1.5 × 10^-10

print(type(price))           # <class 'float'>
```

### ⚠️ Floating Point Precision Warning

This is a classic gotcha that trips up beginners:

```python
print(0.1 + 0.2)
# Expected: 0.3
# Actual:   0.30000000000000004
```

**Why?** Computers store numbers in binary. `0.1` cannot be represented exactly in binary, similar to how `1/3 = 0.333...` cannot be written exactly in decimal.

**Solutions:**

```python
# Option 1: Round for display
result = 0.1 + 0.2
print(round(result, 2))      # 0.3

# Option 2: Use decimal module for financial calculations
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))  # 0.3  — exact!

# Option 3: Compare with tolerance (in practice)
a = 0.1 + 0.2
b = 0.3
print(abs(a - b) < 1e-9)     # True — "close enough"
```

> **In Data Science:** Pandas and NumPy handle float precision gracefully for most tasks. Just be aware of this when doing financial math.

---

## 📝 Strings (`str`)

Strings are **sequences of characters** — any text.

```python
# Creating strings — single, double, or triple quotes
name = "Alice"
city = 'New York'
message = """This is a
multi-line
string."""

# Strings are immutable — you can't change characters in place
greeting = "Hello"
# greeting[0] = "J"  # This would raise an error!
```

### String Operations

```python
first = "Data"
second = "Science"

# Concatenation
combined = first + " " + second
print(combined)                     # "Data Science"

# Repetition
line = "-" * 20
print(line)                         # "--------------------"

# Length
print(len("Hello"))                 # 5

# Indexing (0-based!)
word = "Python"
print(word[0])                      # 'P'
print(word[5])                      # 'n'
print(word[-1])                     # 'n' (last character)
print(word[-2])                     # 'o' (second to last)

# Slicing [start:stop:step]
print(word[0:3])                    # 'Pyt'
print(word[2:])                     # 'thon'
print(word[:4])                     # 'Pyth'
print(word[::2])                    # 'Pto' (every 2nd character)
print(word[::-1])                   # 'nohtyP' (reversed!)
```

### Essential String Methods

```python
text = "  Hello, World!  "

# Case
print(text.upper())                 # "  HELLO, WORLD!  "
print(text.lower())                 # "  hello, world!  "
print(text.title())                 # "  Hello, World!  "
print(text.capitalize())            # "  hello, world!  "

# Whitespace
print(text.strip())                 # "Hello, World!"
print(text.lstrip())                # "Hello, World!  "
print(text.rstrip())                # "  Hello, World!"

# Search
print(text.find("World"))           # 9  (index of first match)
print("World" in text)              # True
print(text.count("l"))              # 3

# Replace
print(text.replace("World", "Python"))  # "  Hello, Python!  "

# Split / Join
sentence = "apple,banana,cherry"
fruits = sentence.split(",")        # ['apple', 'banana', 'cherry']
print(fruits)

joined = " | ".join(fruits)         # 'apple | banana | cherry'
print(joined)

# Check content
print("123".isdigit())              # True
print("abc".isalpha())              # True
print("abc123".isalnum())           # True
print("   ".isspace())              # True
```

### String Formatting — The Modern Way (f-strings)

```python
name = "Alice"
age = 30
score = 95.678

# f-string (Python 3.6+) — RECOMMENDED
print(f"Name: {name}, Age: {age}")           # Name: Alice, Age: 30
print(f"Score: {score:.2f}")                  # Score: 95.68
print(f"Score: {score:.0f}%")                 # Score: 96%
print(f"{name!upper}")                        # ALICE — calling method inside f-string

# Padding and alignment
print(f"{'Left':<10}|")                       # "Left      |"
print(f"{'Right':>10}|")                      # "     Right|"
print(f"{'Center':^10}|")                     # "  Center  |"

# Thousands separator
big_number = 1234567.89
print(f"{big_number:,.2f}")                   # 1,234,567.89

# Older formatting styles (you'll see these in legacy code)
print("Name: %s, Age: %d" % (name, age))      # Old style
print("Name: {}, Age: {}".format(name, age))  # .format() style
```

---

## ✅ Booleans (`bool`)

Booleans have exactly **two possible values**: `True` or `False`.

```python
is_raining = True
is_sunny = False

print(type(is_raining))            # <class 'bool'>
print(True + True)                 # 2 (True = 1, False = 0 in arithmetic)
print(True * 5)                    # 5
print(False * 100)                 # 0
```

### Boolean Operators

```python
# and — both must be True
print(True and True)               # True
print(True and False)              # False
print(False and True)              # False

# or — at least one must be True
print(True or False)               # True
print(False or False)              # False

# not — inverts
print(not True)                    # False
print(not False)                   # True
```

### Comparison Operators (return booleans)

```python
x = 10
y = 20

print(x == y)      # False  — equal to
print(x != y)      # True   — not equal to
print(x < y)       # True   — less than
print(x > y)       # False  — greater than
print(x <= y)      # True   — less than or equal
print(x >= y)      # False  — greater than or equal

# Chaining comparisons (Pythonic!)
age = 25
print(18 <= age <= 65)             # True — age is between 18 and 65
```

### Truthy and Falsy Values

In Python, non-boolean values are evaluated as `True` or `False` in a boolean context:

```python
# These are FALSY (evaluate to False)
bool(0)            # False
bool(0.0)          # False
bool("")           # False — empty string
bool([])           # False — empty list
bool({})           # False — empty dict
bool(None)         # False

# Everything else is TRUTHY
bool(1)            # True
bool(-1)           # True
bool("hello")      # True
bool([1, 2, 3])    # True
```

**Real-world use:**
```python
user_name = ""   # came from a form

if user_name:
    print(f"Hello, {user_name}!")
else:
    print("Please enter your name.")   # This runs — empty string is falsy
```

---

## 🚫 None (`NoneType`)

`None` represents the **absence of a value** — like "nothing" or "missing".

```python
result = None

print(result)                      # None
print(type(result))                # <class 'NoneType'>

# CORRECT way to check for None
if result is None:
    print("No result yet")

# Also correct
if result is not None:
    print(f"Result: {result}")

# AVOID (but works)
if result == None:
    print("No result")
```

> **In Data Science:** `None` is the Python equivalent of `NaN` (Not a Number) in Pandas. Missing values in datasets are often represented as `None` before loading into a DataFrame.

---

## 🔄 Type Conversion

Python lets you convert between types using built-in functions:

```python
# To integer
int("42")              # 42
int(3.9)               # 3 — truncates (does NOT round)
int(True)              # 1
int(False)             # 0

# To float
float("3.14")          # 3.14
float(42)              # 42.0
float("1e5")           # 100000.0

# To string
str(42)                # "42"
str(3.14)              # "3.14"
str(True)              # "True"

# To boolean
bool(1)                # True
bool(0)                # False
bool("hello")          # True
bool("")               # False
```

### Type Conversion Errors

```python
# These will raise ValueError
int("hello")           # ValueError: invalid literal for int()
float("abc")           # ValueError: could not convert string to float

# Safe conversion pattern
def safe_int(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        return None

print(safe_int("42"))       # 42
print(safe_int("hello"))    # None
```

---

## 🔍 Checking Types with `type()` and `isinstance()`

```python
x = 42
y = 3.14
z = "hello"
a = True

# type() — exact type
print(type(x))                     # <class 'int'>
print(type(y))                     # <class 'float'>
print(type(z))                     # <class 'str'>

# type comparison
print(type(x) == int)              # True
print(type(x) == float)            # False

# isinstance() — recommended, handles inheritance
print(isinstance(x, int))          # True
print(isinstance(y, float))        # True
print(isinstance(z, str))          # True
print(isinstance(a, bool))         # True
print(isinstance(a, int))          # True! (bool is a subclass of int)
```

---

## 🔄 Dynamic Typing — Python's Superpower

Python is **dynamically typed** — variables can change types:

```python
x = 42           # x is int
print(type(x))   # <class 'int'>

x = "hello"      # x is now str (no error!)
print(type(x))   # <class 'str'>

x = [1, 2, 3]   # x is now list
print(type(x))   # <class 'list'>
```

Compare to **Java** (statically typed):
```java
int x = 42;
x = "hello";    // ERROR! You declared x as int
```

> **In Data Science:** Dynamic typing is convenient for exploration, but can cause bugs in production. Always validate input types when writing reusable functions.

---

## 🧪 Multiple Assignment & Swapping

```python
# Assign multiple variables at once
x, y, z = 1, 2, 3
print(x, y, z)             # 1 2 3

# Assign same value to multiple variables
a = b = c = 0
print(a, b, c)             # 0 0 0

# Swap values — Pythonic one-liner!
x, y = 10, 20
x, y = y, x
print(x, y)                # 20 10

# Unpack from a list
coordinates = [40.7128, -74.0060]   # NYC
lat, lon = coordinates
print(lat, lon)             # 40.7128 -74.006
```

---

## 📊 Data Types in Data Science — Quick Reference

| Type | Python | Pandas | Use Case |
|------|--------|--------|----------|
| Whole numbers | `int` | `int64` | Counts, IDs, ages |
| Decimals | `float` | `float64` | Prices, measurements, probabilities |
| Text | `str` | `object` | Names, categories, text data |
| Yes/No | `bool` | `bool` | Flags, binary features |
| Missing | `None` | `NaN` | Missing values |

---

## ✅ Key Takeaways

- Variables are **named containers** for values — choose descriptive names
- Python has 4 main scalar types: **int, float, str, bool**
- **None** represents the absence of a value (important in Data Science!)
- Python is **dynamically typed** — variables can change types
- Use **f-strings** for string formatting (modern, readable, fast)
- Watch out for **floating point precision** in financial calculations
- `isinstance()` is preferred over `type()` for type checking

---

## 🔗 What's Next?

➡️ [[03-control-flow]] — Make decisions and repeat actions with if/elif/else and loops