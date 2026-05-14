# 🔧 04 – Functions

> **Prerequisites:** [[03-control-flow]]  
> **Time to read:** ~30 minutes

---

## 🧠 What is a Function?

A **function** is a named, reusable block of code that performs a specific task.

Think of a function like a **recipe**:
- You name it (e.g., `bake_cake`)
- You define what ingredients it needs (parameters)
- It performs steps (the function body)
- It produces a result (the return value)

### Why Use Functions?

```
Without functions:                     With functions:
──────────────────                     ──────────────
# Calculate tax for order 1            def calculate_tax(price, rate=0.18):
price1 = 100                               return price * rate
tax1 = price1 * 0.18
total1 = price1 + tax1                 # Calculate tax for any order
                                       total1 = 100 + calculate_tax(100)
# Calculate tax for order 2            total2 = 250 + calculate_tax(250)
price2 = 250                           total3 = 75  + calculate_tax(75, 0.05)
tax2 = price2 * 0.18
total2 = price2 + tax2

# Calculate tax for order 3
price3 = 75
tax3 = price3 * 0.05                  # Change tax logic? Edit ONE place.
total3 = price3 + tax3
```

**The 3 core benefits:**
1. **DRY** (Don't Repeat Yourself) — write logic once, use many times
2. **Readability** — `calculate_tax(price)` is clearer than raw math
3. **Testability** — test the function once, trust it everywhere

---

## 📝 Defining and Calling Functions

### Basic Syntax

```python
def function_name(parameter1, parameter2):
    """Docstring: explains what the function does."""
    # function body
    return result
```

### Your First Function

```python
# Define the function
def greet(name):
    """Return a personalized greeting."""
    return f"Hello, {name}!"

# Call the function
message = greet("Alice")
print(message)   # Hello, Alice!

print(greet("Bob"))     # Hello, Bob!
print(greet("Charlie")) # Hello, Charlie!
```

### Function Without a Return Value

```python
def print_separator(char="-", length=40):
    """Print a visual separator line."""
    print(char * length)

print_separator()            # ----------------------------------------
print_separator("=")         # ========================================
print_separator("*", 20)     # ********************
```

> When a function has no `return` statement, it implicitly returns `None`.

```python
result = print_separator()
print(result)   # None
```

---

## 📥 Function Parameters

### Required (Positional) Parameters

```python
def add(a, b):
    return a + b

print(add(3, 5))    # 8
print(add(10, 20))  # 30
# print(add(3))     # TypeError: missing required argument 'b'
```

### Default Parameters

```python
def power(base, exponent=2):
    """Raise base to the given exponent. Default is squaring."""
    return base ** exponent

print(power(3))        # 9  (3^2, exponent defaults to 2)
print(power(3, 3))     # 27 (3^3)
print(power(2, 10))    # 1024 (2^10)
```

> ⚠️ **Rule:** Parameters with defaults must come **after** required parameters.
> ```python
> def func(required, optional=10):   # ✅ Correct
> def func(optional=10, required):   # ❌ SyntaxError
> ```

### Keyword Arguments

```python
def describe_person(name, age, city):
    return f"{name} is {age} years old and lives in {city}."

# Positional — order matters
print(describe_person("Alice", 30, "Mumbai"))

# Keyword — order doesn't matter!
print(describe_person(age=30, city="Mumbai", name="Alice"))

# Mix: positional first, then keyword
print(describe_person("Alice", city="Mumbai", age=30))
```

### `*args` — Accept Any Number of Positional Arguments

```python
def add_all(*numbers):
    """Add any number of values."""
    total = 0
    for n in numbers:
        total += n
    return total

print(add_all(1, 2))              # 3
print(add_all(1, 2, 3, 4, 5))    # 15
print(add_all(10, 20, 30))        # 60

# *args is a TUPLE inside the function
def show_args(*args):
    print(type(args))   # <class 'tuple'>
    print(args)
```

### `**kwargs` — Accept Any Number of Keyword Arguments

```python
def describe(**attributes):
    """Describe anything with keyword attributes."""
    for key, value in attributes.items():
        print(f"  {key}: {value}")

describe(name="Alice", age=30, city="Mumbai", job="Data Scientist")
# Output:
#   name: Alice
#   age: 30
#   city: Mumbai
#   job: Data Scientist

# **kwargs is a DICT inside the function
def show_kwargs(**kwargs):
    print(type(kwargs))   # <class 'dict'>
```

### Combining All Parameter Types

```python
def mixed_function(required, default=10, *args, **kwargs):
    print(f"required: {required}")
    print(f"default: {default}")
    print(f"args: {args}")
    print(f"kwargs: {kwargs}")

mixed_function("hello", 20, 1, 2, 3, name="Alice", city="Delhi")
# required: hello
# default: 20
# args: (1, 2, 3)
# kwargs: {'name': 'Alice', 'city': 'Delhi'}
```

---

## 📤 Return Values

### Returning Multiple Values

```python
def min_max(numbers):
    """Return both minimum and maximum of a list."""
    return min(numbers), max(numbers)   # returns a tuple

low, high = min_max([3, 1, 7, 2, 8, 4])
print(f"Min: {low}, Max: {high}")    # Min: 1, Max: 8

# Or capture as one tuple
result = min_max([3, 1, 7, 2, 8, 4])
print(result)                         # (1, 8)
```

### Early Return

```python
def divide(a, b):
    """Divide a by b safely."""
    if b == 0:
        return None   # early return — avoid division by zero
    return a / b

print(divide(10, 2))    # 5.0
print(divide(10, 0))    # None
```

### Returning Dictionaries (Common in Data Science)

```python
def compute_stats(numbers):
    """Compute basic statistics for a list of numbers."""
    n = len(numbers)
    total = sum(numbers)
    mean = total / n
    sorted_nums = sorted(numbers)
    median = sorted_nums[n // 2] if n % 2 != 0 else (sorted_nums[n//2 - 1] + sorted_nums[n//2]) / 2

    return {
        "count": n,
        "sum": total,
        "mean": mean,
        "min": min(numbers),
        "max": max(numbers),
        "median": median,
    }

data = [4, 7, 2, 9, 1, 5, 8, 3, 6]
stats = compute_stats(data)

for key, value in stats.items():
    print(f"  {key:10}: {value}")
```

---

## 📚 Docstrings — Document Your Functions

A **docstring** is a string at the top of a function that explains what it does. It's critical for collaboration and Data Science work.

```python
def clean_text(text, lowercase=True, remove_punctuation=True):
    """
    Clean a text string for NLP preprocessing.

    Args:
        text (str): The raw input text to clean.
        lowercase (bool): If True, convert to lowercase. Default: True.
        remove_punctuation (bool): If True, remove punctuation. Default: True.

    Returns:
        str: The cleaned text string.

    Examples:
        >>> clean_text("Hello, World!")
        'hello world'
        >>> clean_text("Hello, World!", lowercase=False)
        'Hello World'
    """
    if lowercase:
        text = text.lower()
    if remove_punctuation:
        import string
        text = text.translate(str.maketrans("", "", string.punctuation))
    return text.strip()

# Access the docstring
help(clean_text)
print(clean_text.__doc__)
```

---

## ⚡ Lambda Functions — Anonymous One-Liners

A **lambda** is a small, anonymous function written in one line.

```python
# Regular function
def square(x):
    return x ** 2

# Lambda equivalent
square = lambda x: x ** 2

print(square(5))   # 25
```

### Lambdas Are Most Useful With `map()`, `filter()`, `sorted()`

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# map() — apply function to each element
squares = list(map(lambda x: x ** 2, numbers))
print(squares)   # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# filter() — keep elements where function returns True
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)     # [2, 4, 6, 8, 10]

# sorted() — custom sort key
students = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]

# Sort by score (second element)
ranked = sorted(students, key=lambda s: s[1], reverse=True)
print(ranked)    # [('Bob', 92), ('Alice', 85), ('Charlie', 78)]

# Sort strings by length
words = ["banana", "fig", "apple", "date"]
by_length = sorted(words, key=lambda w: len(w))
print(by_length)  # ['fig', 'date', 'apple', 'banana']
```

### In Pandas (Very Common!)

```python
import pandas as pd

df = pd.DataFrame({"salary": [50000, 80000, 120000, 45000]})

# Apply a lambda to a column
df["salary_category"] = df["salary"].apply(
    lambda x: "High" if x > 100000 else ("Mid" if x > 60000 else "Low")
)
print(df)
```

---

## 🔍 Scope — Where Variables Live

Python has a concept called **scope**: the region of code where a variable is accessible.

### LEGB Rule

Python looks up variables in this order:
1. **L**ocal — inside the current function
2. **E**nclosing — in any enclosing function
3. **G**lobal — at the module level
4. **B**uilt-in — Python's built-in names

```python
x = "global"   # Global scope

def outer():
    x = "enclosing"   # Enclosing scope

    def inner():
        x = "local"   # Local scope
        print(x)      # "local"

    inner()
    print(x)          # "enclosing"

outer()
print(x)              # "global"
```

### The `global` and `nonlocal` Keywords

```python
count = 0   # global variable

def increment():
    global count   # explicitly reference the global variable
    count += 1

increment()
increment()
print(count)   # 2
```

> **Best practice:** Avoid `global` variables in production code — they make functions hard to test and reason about. Pass values as arguments and return them instead.

---

## 🧬 First-Class Functions — Functions as Values

In Python, functions are **first-class citizens** — they can be stored in variables, passed to other functions, and returned from functions.

```python
# Storing a function in a variable
def greet(name):
    return f"Hello, {name}!"

say_hello = greet   # no parentheses — we're copying the function, not calling it
print(say_hello("Alice"))   # Hello, Alice!

# Passing a function as an argument
def apply_twice(func, value):
    return func(func(value))

def double(x):
    return x * 2

print(apply_twice(double, 3))   # 12  (double(double(3)) = double(6) = 12)
```

### Higher-Order Functions in Data Science

```python
# A function that returns different "cleaning" functions
def make_outlier_remover(threshold):
    """Return a function that removes values above threshold."""
    def remove_outliers(data):
        return [x for x in data if x <= threshold]
    return remove_outliers

# Create specialized functions
remove_above_100 = make_outlier_remover(100)
remove_above_50  = make_outlier_remover(50)

data = [10, 20, 150, 30, 200, 40]
print(remove_above_100(data))   # [10, 20, 30, 40]
print(remove_above_50(data))    # [10, 20, 30, 40]
```

---

## 🧩 Real Data Science Function Examples

### Example 1: Data Validator

```python
def validate_age(age):
    """Validate age input and return cleaned value or raise error."""
    if not isinstance(age, (int, float)):
        raise TypeError(f"Age must be numeric, got {type(age).__name__}")
    if age < 0 or age > 150:
        raise ValueError(f"Age {age} is out of realistic range (0–150)")
    return int(age)

# Usage with error handling
test_ages = [25, -5, 200, "thirty", 45.7]
for a in test_ages:
    try:
        clean = validate_age(a)
        print(f"✅ {a} → {clean}")
    except (TypeError, ValueError) as e:
        print(f"❌ {a} → Error: {e}")
```

### Example 2: Feature Engineering Helper

```python
def bin_values(value, bins, labels):
    """
    Map a numeric value to a category label.

    Args:
        value (float): The value to bin.
        bins (list): Sorted list of threshold values.
        labels (list): Labels for each bin (len = len(bins) + 1).

    Returns:
        str: The category label.
    """
    for i, threshold in enumerate(bins):
        if value <= threshold:
            return labels[i]
    return labels[-1]

# Categorize income
income_bins   = [25000, 50000, 100000, 200000]
income_labels = ["Very Low", "Low", "Middle", "High", "Very High"]

test_incomes = [15000, 35000, 75000, 150000, 300000]
for income in test_incomes:
    category = bin_values(income, income_bins, income_labels)
    print(f"${income:>8,} → {category}")
```

### Example 3: Summary Statistics

```python
def summarize(data, name="Dataset"):
    """Print a statistical summary of a numeric list."""
    if not data:
        print(f"{name}: Empty dataset")
        return

    n = len(data)
    total = sum(data)
    mean = total / n
    variance = sum((x - mean) ** 2 for x in data) / n
    std = variance ** 0.5
    sorted_data = sorted(data)

    print(f"{'=' * 30}")
    print(f"  Summary: {name}")
    print(f"{'=' * 30}")
    print(f"  Count:  {n}")
    print(f"  Sum:    {total:.2f}")
    print(f"  Mean:   {mean:.2f}")
    print(f"  Std:    {std:.2f}")
    print(f"  Min:    {sorted_data[0]}")
    print(f"  Max:    {sorted_data[-1]}")
    print(f"{'=' * 30}")

scores = [85, 92, 78, 61, 45, 90, 88, 73, 55, 67]
summarize(scores, "Exam Scores")
```

---

## ✅ Key Takeaways

- Functions make code **reusable, readable, and testable**
- Use **default parameters** for optional inputs
- `*args` collects extra positional args as a **tuple**; `**kwargs` as a **dict**
- Always write **docstrings** — your future self will thank you
- **Lambda** functions are concise one-liners, great for `map()`, `filter()`, and Pandas `.apply()`
- Functions are **first-class objects** — they can be passed around like any value
- Understand **scope** (LEGB rule) to avoid subtle bugs

---

## 🔗 What's Next?

➡️ [[05-lists-tuples-dictionaries]] — Python's core data structures