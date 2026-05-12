# 📚 05 – Lists, Tuples & Dictionaries

> **Prerequisites:** [[04-functions]]  
> **Time to read:** ~30 minutes

---

## 🗂️ Python's Core Data Structures

A **data structure** is a way of organizing multiple values. Python has four built-in collection types:

| Structure | Syntax | Ordered | Mutable | Duplicates |
|-----------|--------|---------|---------|------------|
| **List** | `[1, 2, 3]` | ✅ Yes | ✅ Yes | ✅ Yes |
| **Tuple** | `(1, 2, 3)` | ✅ Yes | ❌ No | ✅ Yes |
| **Dictionary** | `{"key": "val"}` | ✅ Yes (3.7+) | ✅ Yes | Keys: No |
| **Set** | `{1, 2, 3}` | ❌ No | ✅ Yes | ❌ No |

> **Mutable** means you can change the contents after creation.

---

## 📋 Lists

A **list** is an ordered, mutable collection. It's Python's most versatile data structure and the backbone of most data processing.

### Creating Lists

```python
# Empty list
empty = []
also_empty = list()

# With values
numbers = [1, 2, 3, 4, 5]
names = ["Alice", "Bob", "Charlie"]

# Mixed types (possible, but usually a sign of bad design)
mixed = [1, "hello", 3.14, True, None]

# Nested lists (like a 2D matrix)
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# List from range
first_10 = list(range(1, 11))       # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# List from string
chars = list("Python")              # ['P', 'y', 't', 'h', 'o', 'n']
```

### Indexing and Slicing

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]
#          0        1         2         3       4
#         -5       -4        -3        -2      -1

# Indexing
print(fruits[0])      # "apple"       (first)
print(fruits[2])      # "cherry"      (third)
print(fruits[-1])     # "elderberry"  (last)
print(fruits[-2])     # "date"        (second to last)

# Slicing [start:stop:step]   (stop is exclusive!)
print(fruits[1:3])    # ['banana', 'cherry']
print(fruits[:3])     # ['apple', 'banana', 'cherry']  (first 3)
print(fruits[2:])     # ['cherry', 'date', 'elderberry']
print(fruits[::2])    # ['apple', 'cherry', 'elderberry']  (every 2nd)
print(fruits[::-1])   # ['elderberry', 'date', 'cherry', 'banana', 'apple'] (reversed)

# Copy a list via slicing
copy = fruits[:]      # Creates a new list with same content
```

### Modifying Lists

```python
colors = ["red", "green", "blue"]

# Change an element
colors[1] = "yellow"
print(colors)             # ['red', 'yellow', 'blue']

# append — add to the end
colors.append("purple")
print(colors)             # ['red', 'yellow', 'blue', 'purple']

# insert — add at specific position
colors.insert(1, "orange")
print(colors)             # ['red', 'orange', 'yellow', 'blue', 'purple']

# extend — add multiple items
colors.extend(["pink", "brown"])
print(colors)             # ['red', 'orange', 'yellow', 'blue', 'purple', 'pink', 'brown']

# remove — remove first occurrence of value
colors.remove("yellow")
print(colors)             # ['red', 'orange', 'blue', 'purple', 'pink', 'brown']

# pop — remove and return item (last by default)
last = colors.pop()
print(last)               # 'brown'
first = colors.pop(0)
print(first)              # 'red'

# del — delete by index or slice
del colors[1]             # removes 'blue'
print(colors)             # ['orange', 'purple', 'pink']

# clear — remove all items
# colors.clear()          # would make colors = []
```

### Essential List Methods

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]

# Information
print(len(numbers))                 # 11
print(numbers.count(5))             # 3  (5 appears 3 times)
print(numbers.index(9))             # 5  (first index of value 9)
print(min(numbers))                 # 1
print(max(numbers))                 # 9
print(sum(numbers))                 # 44

# Sorting
numbers.sort()                      # Sorts IN PLACE (modifies original)
print(numbers)                      # [1, 1, 2, 3, 3, 4, 5, 5, 5, 6, 9]

numbers.sort(reverse=True)          # Descending
print(numbers)                      # [9, 6, 5, 5, 5, 4, 3, 3, 2, 1, 1]

original = [3, 1, 4, 1, 5, 9]
sorted_copy = sorted(original)      # Returns NEW list, original unchanged
print(original)                     # [3, 1, 4, 1, 5, 9]
print(sorted_copy)                  # [1, 1, 3, 4, 5, 9]

# Reverse
numbers.reverse()                   # Reverses IN PLACE
print(list(reversed(original)))     # Returns new reversed iterator
```

### Common List Patterns

```python
# Check membership
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)           # True
print("grape" in fruits)            # False
print("grape" not in fruits)        # True

# Flatten a nested list
nested = [[1, 2], [3, 4], [5, 6]]
flat = [x for sublist in nested for x in sublist]
print(flat)                         # [1, 2, 3, 4, 5, 6]

# Remove duplicates while preserving order
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
seen = set()
unique = []
for x in data:
    if x not in seen:
        unique.append(x)
        seen.add(x)
print(unique)   # [3, 1, 4, 5, 9, 2, 6]

# Or using dict.fromkeys (preserves order, Python 3.7+)
unique = list(dict.fromkeys(data))
print(unique)   # [3, 1, 4, 5, 9, 2, 6]

# Zip two lists into pairs
keys = ["a", "b", "c"]
vals = [1, 2, 3]
pairs = list(zip(keys, vals))       # [('a', 1), ('b', 2), ('c', 3)]
```

### List Comprehension (Advanced)

```python
# Filter and transform in one line
numbers = range(1, 21)

# Squares of even numbers
even_squares = [x**2 for x in numbers if x % 2 == 0]
print(even_squares)   # [4, 16, 36, 64, 100, 144, 196, 256, 324, 400]

# Nested comprehension: flatten 2D to 1D
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [cell for row in matrix for cell in row]
print(flat)   # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Dictionary comprehension
squared = {x: x**2 for x in range(1, 6)}
print(squared)   # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

---

## 📌 Tuples

A **tuple** is an ordered, **immutable** (unchangeable) collection.

### When to Use Tuples vs Lists

| Use Tuple | Use List |
|-----------|----------|
| Data that shouldn't change (coordinates, RGB colors) | Data that will grow or change |
| Function return values | Items to be sorted or filtered |
| Dictionary keys | Accumulating results |
| Fixed configurations | Any dynamic collection |

```python
# Creating tuples
point = (3, 7)
rgb_red = (255, 0, 0)
empty_tuple = ()
single_item = (42,)          # IMPORTANT: trailing comma for single-element tuple
also_tuple = 1, 2, 3         # parentheses optional!

# Indexing and slicing — same as lists
print(point[0])               # 3
print(rgb_red[-1])            # 0

# Immutability — this raises an error
try:
    point[0] = 5
except TypeError as e:
    print(f"Error: {e}")      # Error: 'tuple' object does not support item assignment
```

### Tuple Unpacking

```python
# Basic unpacking
x, y = (10, 20)
print(x, y)                   # 10 20

# Swap variables (uses tuple packing/unpacking)
a, b = 5, 10
a, b = b, a
print(a, b)                   # 10 5

# Unpack with *rest
first, *rest = (1, 2, 3, 4, 5)
print(first)                  # 1
print(rest)                   # [2, 3, 4, 5]

first, *middle, last = (1, 2, 3, 4, 5)
print(first, middle, last)    # 1 [2, 3, 4] 5

# Ignore values with _
_, important, _ = ("ignore", "keep this", "also ignore")
print(important)              # "keep this"

# From function return
def get_coordinates():
    return 40.7128, -74.0060   # returns a tuple

lat, lon = get_coordinates()
print(f"Lat: {lat}, Lon: {lon}")
```

### Named Tuples — Best of Both Worlds

```python
from collections import namedtuple

# Define a named tuple type
Point = namedtuple("Point", ["x", "y"])
Person = namedtuple("Person", ["name", "age", "city"])

# Create instances
p = Point(3, 7)
alice = Person("Alice", 30, "Mumbai")

# Access by name OR index
print(p.x, p.y)                    # 3 7
print(p[0], p[1])                  # 3 7

print(alice.name)                  # Alice
print(alice.age)                   # 30

# Still immutable
# alice.age = 31  # AttributeError
```

---

## 📖 Dictionaries

A **dictionary** is a collection of **key-value pairs** — like a real dictionary where you look up a word (key) to get its definition (value).

### Creating Dictionaries

```python
# Empty dictionary
empty = {}
also_empty = dict()

# With data
person = {
    "name": "Alice",
    "age": 30,
    "city": "Mumbai",
    "is_employed": True
}

# From keyword arguments
config = dict(host="localhost", port=5432, database="mydb")

# From list of tuples
pairs = [("a", 1), ("b", 2), ("c", 3)]
d = dict(pairs)
print(d)    # {'a': 1, 'b': 2, 'c': 3}
```

### Accessing Values

```python
person = {"name": "Alice", "age": 30, "city": "Mumbai"}

# Direct access (KeyError if key doesn't exist)
print(person["name"])             # Alice

# Safe access with .get() (returns None if key missing)
print(person.get("age"))          # 30
print(person.get("email"))        # None
print(person.get("email", "N/A")) # N/A  — custom default!

# Check if key exists
print("name" in person)           # True
print("email" in person)          # False
```

### Modifying Dictionaries

```python
person = {"name": "Alice", "age": 30}

# Add / update
person["email"] = "alice@example.com"   # Add new key
person["age"] = 31                       # Update existing key

# Update multiple keys at once
person.update({"city": "Delhi", "age": 32})

# Remove
del person["email"]                      # Raises KeyError if not found

removed = person.pop("city")             # Remove and return value
print(removed)                           # Delhi

# Remove last inserted item
last = person.popitem()                  # Returns (key, value) tuple
print(last)                              # ('age', 32)
```

### Iterating Over Dictionaries

```python
scores = {"Alice": 85, "Bob": 92, "Charlie": 78, "Diana": 95}

# Iterate over keys (default)
for name in scores:
    print(name)                          # Alice, Bob, Charlie, Diana

# Iterate over values
for score in scores.values():
    print(score)                         # 85, 92, 78, 95

# Iterate over key-value pairs — MOST COMMON
for name, score in scores.items():
    grade = "A" if score >= 90 else "B" if score >= 80 else "C"
    print(f"{name}: {score} ({grade})")

# Get keys/values as lists
keys = list(scores.keys())              # ['Alice', 'Bob', 'Charlie', 'Diana']
values = list(scores.values())          # [85, 92, 78, 95]
```

### Dictionary Comprehension

```python
# Square numbers dict
squares = {x: x**2 for x in range(1, 6)}
print(squares)    # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Filter students above average
scores = {"Alice": 85, "Bob": 92, "Charlie": 60, "Diana": 75}
avg = sum(scores.values()) / len(scores)
above_avg = {name: s for name, s in scores.items() if s > avg}
print(above_avg)  # {'Alice': 85, 'Bob': 92}

# Invert a dictionary (swap keys and values)
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)   # {1: 'a', 2: 'b', 3: 'c'}
```

### Nested Dictionaries

```python
# Representing a dataset record
employee = {
    "id": 1001,
    "name": "Alice",
    "department": "Data Science",
    "contact": {
        "email": "alice@company.com",
        "phone": "+91-9876543210"
    },
    "skills": ["Python", "SQL", "Machine Learning"],
    "performance": {
        "2023": {"score": 4.2, "rating": "Exceeds"},
        "2024": {"score": 4.5, "rating": "Outstanding"}
    }
}

# Deep access
print(employee["contact"]["email"])                # alice@company.com
print(employee["skills"][0])                       # Python
print(employee["performance"]["2024"]["rating"])   # Outstanding

# Safe deep access
rating = employee.get("performance", {}).get("2025", {}).get("rating", "Not evaluated")
print(rating)   # "Not evaluated"
```

---

## 🔧 Sets (Bonus)

A **set** is an unordered collection of **unique** items.

```python
# Creating sets
fruits = {"apple", "banana", "cherry", "apple"}   # duplicate removed!
print(fruits)    # {'apple', 'banana', 'cherry'} (order may vary)

# From a list
numbers = list({3, 1, 4, 1, 5, 9, 2, 6, 5, 3})
print(numbers)   # [1, 2, 3, 4, 5, 6, 9]  (unique, unordered)

# Set operations — very useful in Data Science!
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

print(a | b)     # Union: {1, 2, 3, 4, 5, 6, 7, 8}
print(a & b)     # Intersection: {4, 5}
print(a - b)     # Difference: {1, 2, 3}
print(a ^ b)     # Symmetric diff: {1, 2, 3, 6, 7, 8}

# Fast membership check (O(1) vs O(n) for list)
big_set = set(range(1_000_000))
print(999999 in big_set)   # True — instant!
```

---

## 🧩 Data Science Examples

### Example 1: Word Frequency Counter

```python
text = "the cat sat on the mat the cat ate the rat"
words = text.split()

# Count word frequencies using a dict
freq = {}
for word in words:
    freq[word] = freq.get(word, 0) + 1

# Sort by frequency
sorted_freq = sorted(freq.items(), key=lambda x: x[1], reverse=True)
for word, count in sorted_freq:
    print(f"  {word:10}: {count}")
```

### Example 2: Data Cleaning Pipeline

```python
# Raw customer data (messy)
raw_records = [
    {"name": "alice", "age": "29", "salary": "50000"},
    {"name": "BOB", "age": "35", "salary": "invalid"},
    {"name": "  Charlie  ", "age": "42", "salary": "80000"},
    {"name": "diana", "age": "-5", "salary": "65000"},
]

def clean_record(record):
    """Clean a single record dict."""
    try:
        age = int(record["age"])
        if age < 0 or age > 120:
            raise ValueError("Invalid age")
    except ValueError:
        age = None

    try:
        salary = float(record["salary"])
    except ValueError:
        salary = None

    return {
        "name": record["name"].strip().title(),
        "age": age,
        "salary": salary,
        "is_valid": age is not None and salary is not None
    }

cleaned = [clean_record(r) for r in raw_records]
for r in cleaned:
    print(r)
```

### Example 3: Group Data by Category

```python
# Group students by grade
students = [
    {"name": "Alice", "score": 92},
    {"name": "Bob", "score": 75},
    {"name": "Charlie", "score": 88},
    {"name": "Diana", "score": 60},
    {"name": "Eve", "score": 95},
]

# Group into A/B/C
groups = {"A": [], "B": [], "C": []}

for student in students:
    if student["score"] >= 90:
        groups["A"].append(student["name"])
    elif student["score"] >= 75:
        groups["B"].append(student["name"])
    else:
        groups["C"].append(student["name"])

for grade, names in groups.items():
    print(f"Grade {grade}: {', '.join(names)}")
```

---

## ✅ Key Takeaways

- **Lists** are ordered, mutable sequences — use them for most collections
- **Tuples** are ordered, immutable — use them for fixed data and multiple return values
- **Dictionaries** are key-value stores — use them to represent structured records and look up data by name
- **Sets** are unordered unique collections — use them for membership tests and set operations
- **List comprehensions** and **dict comprehensions** are the Pythonic way to build collections
- Master `enumerate()`, `zip()`, `.items()`, `.get()` — you'll use them in every Data Science project

---

## 🔗 What's Next?

➡️ [[06-practice-problems]] — Solidify your knowledge with hands-on challenges