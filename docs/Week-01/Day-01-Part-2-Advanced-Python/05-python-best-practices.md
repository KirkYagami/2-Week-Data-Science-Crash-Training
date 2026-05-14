# ✨ 05 – Python Best Practices

> **Prerequisites:** [[04-exception-handling]]  
> **Time to read:** ~20 minutes

---

## 🧠 Why Best Practices Matter

Writing code that *works* is step one. Writing code that is *maintainable, readable, and professional* is step two — and it's what separates a beginner from an employable Data Scientist.

> "Code is read far more often than it is written." — Guido van Rossum

---

## 📏 PEP 8 — The Python Style Standard

### Naming Conventions

```python
# Variables and functions: snake_case
student_name = "Alice"
total_score = 95
def calculate_average(scores): ...

# Classes: PascalCase (UpperCamelCase)
class DataProcessor: ...
class LinearRegressionModel: ...

# Constants: UPPER_SNAKE_CASE
MAX_ITERATIONS = 1000
DEFAULT_LEARNING_RATE = 0.01
PI = 3.14159

# "Private" (single underscore = convention only)
_internal_state = []

# Truly private (name-mangled)
class MyClass:
    def __init__(self):
        self.__secret = 42
```

### Spacing

```python
# Around operators: spaces
x = 1 + 2        # ✅
x=1+2            # ❌

# After commas: space
f(a, b, c)       # ✅
f(a,b,c)         # ❌

# No spaces around = in keyword arguments
def func(x=10): ...  # ✅  (no spaces)
func(x=10)           # ✅  (no spaces)

# Max line length: 79-99 characters
# Break long lines:
result = (first_value
          + second_value
          + third_value)
```

### Blank Lines

```python
# 2 blank lines between top-level functions/classes
def function_one():
    pass


def function_two():
    pass


class MyClass:
    # 1 blank line between methods
    def method_one(self):
        pass

    def method_two(self):
        pass
```

---

## 🐍 Pythonic Code — Write Python Like a Pro

### 1. Use `enumerate()` instead of manual index

```python
fruits = ["apple", "banana", "cherry"]

# ❌ Not Pythonic
i = 0
for fruit in fruits:
    print(i, fruit)
    i += 1

# ✅ Pythonic
for i, fruit in enumerate(fruits):
    print(i, fruit)
```

### 2. Use `zip()` to iterate in parallel

```python
names = ["Alice", "Bob"]
scores = [85, 92]

# ❌ Not Pythonic
for i in range(len(names)):
    print(names[i], scores[i])

# ✅ Pythonic
for name, score in zip(names, scores):
    print(name, score)
```

### 3. Use list/dict/set comprehensions

```python
# ❌ Verbose
squares = []
for x in range(10):
    squares.append(x**2)

# ✅ Comprehension
squares = [x**2 for x in range(10)]

# ❌ Verbose dict building
freq = {}
for word in words:
    if word in freq:
        freq[word] += 1
    else:
        freq[word] = 0

# ✅ Use Counter or get()
from collections import Counter
freq = Counter(words)
# OR
freq = {}
for word in words:
    freq[word] = freq.get(word, 0) + 1
```

### 4. Use `in` for membership (not loops)

```python
# ❌ Loop to check membership
found = False
for item in my_list:
    if item == target:
        found = True
        break

# ✅ Direct
found = target in my_list

# For fast lookup of many items, use a set
valid_ids = {101, 102, 103, 201, 202}
if user_id in valid_ids:   # O(1)
    process_user(user_id)
```

### 5. Unpack tuples and return multiple values

```python
# ❌ Using index
point = (3, 7)
x = point[0]
y = point[1]

# ✅ Unpack
x, y = point

# ❌ Return dict to fake multiple returns
def get_range(data):
    return {"min": min(data), "max": max(data)}

# ✅ Return tuple, unpack
def get_range(data):
    return min(data), max(data)

low, high = get_range([3, 1, 7, 2])
```

### 6. Use `with` for resource management

```python
# ❌ Manual close (risky if exception)
f = open("data.txt")
data = f.read()
f.close()

# ✅ Context manager
with open("data.txt") as f:
    data = f.read()
```

### 7. Use f-strings (Python 3.6+)

```python
name = "Alice"
score = 95.678

# ❌ Old style
print("Name: %s, Score: %.2f" % (name, score))

# ❌ .format()
print("Name: {}, Score: {:.2f}".format(name, score))

# ✅ f-string
print(f"Name: {name}, Score: {score:.2f}")
```

### 8. Use `any()` and `all()`

```python
scores = [85, 92, 60, 78]

# Check if any score is failing
has_failure = any(s < 60 for s in scores)

# Check if all scores are passing
all_passing = all(s >= 60 for s in scores)

# ❌ Manual loop
has_failure = False
for s in scores:
    if s < 60:
        has_failure = True
        break
```

---

## 📝 Documentation Best Practices

```python
def calculate_bmi(weight_kg, height_m):
    """
    Calculate Body Mass Index (BMI).

    BMI = weight (kg) / height (m)^2

    Args:
        weight_kg (float): Weight in kilograms. Must be positive.
        height_m (float): Height in meters. Must be positive.

    Returns:
        float: The calculated BMI value.

    Raises:
        ValueError: If weight or height are non-positive.

    Examples:
        >>> calculate_bmi(70, 1.75)
        22.857142857142858
        >>> calculate_bmi(90, 1.80)
        27.777777777777775
    """
    if weight_kg <= 0 or height_m <= 0:
        raise ValueError("Weight and height must be positive numbers")
    return weight_kg / height_m ** 2
```

---

## 🔍 Type Hints (Python 3.5+)

Type hints make code self-documenting and enable IDE autocompletion:

```python
from typing import List, Dict, Optional, Tuple, Union

def process_scores(scores: List[float]) -> Dict[str, float]:
    """Process a list of scores and return statistics."""
    return {
        "mean": sum(scores) / len(scores),
        "min": min(scores),
        "max": max(scores),
    }

def find_user(user_id: int) -> Optional[Dict]:
    """Find a user by ID. Returns None if not found."""
    ...

def normalize(values: List[float], method: str = "minmax") -> List[float]:
    ...
```

---

## 🚫 Common Anti-Patterns to Avoid

```python
# ❌ Using bare except
try:
    risky()
except:          # catches EVERYTHING including SystemExit, KeyboardInterrupt
    pass

# ✅ Always specify the exception
try:
    risky()
except ValueError as e:
    print(f"Error: {e}")


# ❌ Comparing to None with ==
if value == None: ...

# ✅ Use 'is'
if value is None: ...


# ❌ Using mutable default argument
def append_item(item, lst=[]):   # BUG!
    lst.append(item)
    return lst

# ✅ Use None as default
def append_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst


# ❌ Building strings in a loop
result = ""
for s in strings:
    result += s   # creates new string each time — O(n²)

# ✅ Use join
result = "".join(strings)
```

---

## 🔁 Generator Functions — Memory-Efficient Iteration

```python
# List comprehension — loads ALL data into memory
squares = [x**2 for x in range(1_000_000)]  # 8 MB in memory

# Generator — lazy, one item at a time — VERY MEMORY EFFICIENT
def generate_squares(n):
    for x in range(n):
        yield x ** 2

gen = generate_squares(1_000_000)  # barely any memory!
print(next(gen))   # 0
print(next(gen))   # 1

# Generator expression (like list comp but lazy)
gen = (x**2 for x in range(1_000_000))

# Use in for loop
for square in generate_squares(5):
    print(square)   # 0, 1, 4, 9, 16

# In Data Science: great for processing large datasets line by line
def read_large_csv(filepath):
    with open(filepath) as f:
        header = f.readline().strip().split(",")
        for line in f:
            values = line.strip().split(",")
            yield dict(zip(header, values))

for record in read_large_csv("huge_file.csv"):
    process(record)   # only one record in memory at a time!
```

---

## ✅ Key Takeaways

- Follow **PEP 8**: `snake_case` for variables/functions, `PascalCase` for classes, `UPPER_CASE` for constants
- Write **Pythonic code**: use comprehensions, `enumerate`, `zip`, `any`/`all`, unpacking
- Use **f-strings** for formatting (faster and more readable)
- Always write **docstrings** for functions
- Add **type hints** — they improve code quality and IDE support
- Avoid: bare `except`, `== None`, mutable defaults, string concatenation in loops
- Use **generators** for memory-efficient iteration over large datasets

---

## 🔗 What's Next?

➡️ [[06-mini-exercises]] — Practice everything you've learned!