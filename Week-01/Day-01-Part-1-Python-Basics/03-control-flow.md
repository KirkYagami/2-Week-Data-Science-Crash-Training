# 🔀 03 – Control Flow

> **Prerequisites:** [[02-variables-and-data-types]]  
> **Time to read:** ~30 minutes

---

## 🧠 What is Control Flow?

By default, Python runs code **top to bottom, one line at a time**. Control flow lets you:

1. **Branch** — do different things based on conditions (`if/elif/else`)
2. **Repeat** — run the same code multiple times (`for`, `while`)
3. **Skip or exit** — bypass code or escape loops (`break`, `continue`, `pass`)

Without control flow, programs would be useless — they couldn't make decisions or automate repetitive tasks.

---

## 🌿 Conditional Statements — `if / elif / else`

### Basic Syntax

```python
if condition:
    # runs if condition is True
elif another_condition:
    # runs if previous conditions were False AND this is True
else:
    # runs if ALL previous conditions were False
```

> ⚠️ **Indentation matters in Python!** Always use 4 spaces (not tabs) to indent blocks.

### Simple Example

```python
temperature = 28

if temperature > 35:
    print("It's very hot! Stay hydrated.")
elif temperature > 25:
    print("It's warm. Nice weather!")
elif temperature > 15:
    print("It's mild. Bring a light jacket.")
else:
    print("It's cold. Wear warm clothes!")

# Output: "It's warm. Nice weather!"
```

### Data Science Example — Categorizing Data

```python
def categorize_age(age):
    if age < 0:
        return "Invalid age"
    elif age < 13:
        return "Child"
    elif age < 18:
        return "Teenager"
    elif age < 65:
        return "Adult"
    else:
        return "Senior"

print(categorize_age(8))       # Child
print(categorize_age(16))      # Teenager
print(categorize_age(35))      # Adult
print(categorize_age(70))      # Senior
print(categorize_age(-5))      # Invalid age
```

### Nested `if` Statements

```python
score = 85
attendance = 90

if score >= 60:
    if attendance >= 75:
        print("Pass")
    else:
        print("Fail (poor attendance)")
else:
    print("Fail (low score)")

# Output: "Pass"
```

> **Tip:** Avoid deep nesting (more than 2–3 levels). It makes code hard to read. Use `elif` or early returns instead.

### One-Line `if` (Ternary Operator)

```python
# Syntax: value_if_true if condition else value_if_false
x = 10
result = "positive" if x > 0 else "non-positive"
print(result)   # "positive"

# Useful in Data Science for quick transformations
grade = 85
label = "Pass" if grade >= 60 else "Fail"
```

---

## 🔁 `for` Loops — Iterate Over a Sequence

A `for` loop runs a block of code **once for each item** in a sequence.

### Basic Syntax

```python
for variable in sequence:
    # code to run for each item
```

### Iterating Over Lists

```python
fruits = ["apple", "banana", "cherry", "date"]

for fruit in fruits:
    print(fruit)

# Output:
# apple
# banana
# cherry
# date
```

### Iterating Over Strings

```python
word = "Python"
for char in word:
    print(char, end=" ")   # end=" " prevents newline

# Output: P y t h o n
```

### `range()` — Generate a Sequence of Numbers

```python
# range(stop) — from 0 to stop-1
for i in range(5):
    print(i)
# Output: 0 1 2 3 4

# range(start, stop) — from start to stop-1
for i in range(2, 7):
    print(i)
# Output: 2 3 4 5 6

# range(start, stop, step) — with custom step
for i in range(0, 20, 5):
    print(i)
# Output: 0 5 10 15

# Counting down
for i in range(10, 0, -1):
    print(i, end=" ")
# Output: 10 9 8 7 6 5 4 3 2 1
```

### Useful `for` Loop Patterns

#### `enumerate()` — Get index AND value

```python
students = ["Alice", "Bob", "Charlie"]

for index, student in enumerate(students):
    print(f"{index + 1}. {student}")

# Output:
# 1. Alice
# 2. Bob
# 3. Charlie

# Start index at 1
for i, name in enumerate(students, start=1):
    print(f"Student #{i}: {name}")
```

#### `zip()` — Iterate over multiple lists simultaneously

```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
grades = ["B", "A", "C"]

for name, score, grade in zip(names, scores, grades):
    print(f"{name}: {score} ({grade})")

# Output:
# Alice: 85 (B)
# Bob: 92 (A)
# Charlie: 78 (C)
```

#### Nested `for` loops

```python
# Multiplication table
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} × {j} = {i*j}", end="   ")
    print()   # newline after each row

# Output:
# 1 × 1 = 1   1 × 2 = 2   1 × 3 = 3
# 2 × 1 = 2   2 × 2 = 4   2 × 3 = 6
# 3 × 1 = 3   3 × 2 = 6   3 × 3 = 9
```

### List Comprehension — Pythonic Loops

List comprehensions are a **compact, readable way** to build lists. Data Scientists use them constantly.

```python
# Standard for loop
squares = []
for i in range(1, 6):
    squares.append(i ** 2)
print(squares)   # [1, 4, 9, 16, 25]

# List comprehension (same result, one line!)
squares = [i ** 2 for i in range(1, 6)]
print(squares)   # [1, 4, 9, 16, 25]

# With condition (filter)
even_squares = [i ** 2 for i in range(1, 11) if i % 2 == 0]
print(even_squares)   # [4, 16, 36, 64, 100]

# Transform strings
names = ["alice", "bob", "charlie"]
upper_names = [name.upper() for name in names]
print(upper_names)   # ['ALICE', 'BOB', 'CHARLIE']

# Data Science use case: clean a column of prices
prices = ["$10.99", "$25.00", "$5.49"]
clean_prices = [float(p.replace("$", "")) for p in prices]
print(clean_prices)   # [10.99, 25.0, 5.49]
```

---

## 🔄 `while` Loops — Repeat Until a Condition is False

A `while` loop keeps running as long as a condition is **True**.

### Basic Syntax

```python
while condition:
    # code to run
    # (make sure condition eventually becomes False!)
```

### Simple Example

```python
count = 0
while count < 5:
    print(f"Count: {count}")
    count += 1

# Output:
# Count: 0
# Count: 1
# Count: 2
# Count: 3
# Count: 4
```

### ⚠️ Infinite Loops — The Beginner's Nightmare

```python
# DANGER! This loop never ends!
# count = 0
# while count < 5:
#     print(count)
#     # Forgot to increment count!

# Always make sure the condition will eventually be False.
```

### `while` Loop with User Input

```python
while True:
    answer = input("Type 'quit' to exit, or anything else to continue: ")
    if answer.lower() == "quit":
        print("Goodbye!")
        break
    print(f"You typed: {answer}")
```

### Data Science Use Case — Convergence Loop

```python
# Simulating gradient descent convergence
error = 100.0
learning_rate = 0.1
iteration = 0

while error > 0.01:
    error = error * (1 - learning_rate)  # simplified update
    iteration += 1
    print(f"Iteration {iteration}: error = {error:.4f}")

print(f"Converged after {iteration} iterations!")
```

---

## ⏭️ Loop Control — `break`, `continue`, `pass`

### `break` — Exit the Loop Immediately

```python
numbers = [1, 3, 5, 8, 9, 11]

for num in numbers:
    if num % 2 == 0:
        print(f"Found first even number: {num}")
        break   # stop looking

# Output: Found first even number: 8
```

### `continue` — Skip This Iteration

```python
for i in range(10):
    if i % 2 == 0:
        continue   # skip even numbers
    print(i, end=" ")

# Output: 1 3 5 7 9
```

### `pass` — Do Nothing (Placeholder)

```python
# pass is used when you need a block syntactically but have nothing to put there yet
for i in range(5):
    if i == 3:
        pass       # TODO: handle this case later
    else:
        print(i)

# Output: 0 1 2 4
```

### `else` on Loops — Runs if Loop Completed Normally

```python
# else block runs ONLY if the loop was NOT broken out of
target = 7
numbers = [1, 3, 5, 9, 11]

for num in numbers:
    if num == target:
        print(f"Found {target}!")
        break
else:
    print(f"{target} not found in the list.")

# Output: 7 not found in the list.
```

---

## 🧩 Putting It All Together — Real Data Science Examples

### Example 1: Grade Distribution Analyzer

```python
grades = [85, 92, 78, 61, 45, 90, 88, 73, 55, 67]

# Count grade categories
a_count = 0
b_count = 0
c_count = 0
d_count = 0
f_count = 0

for grade in grades:
    if grade >= 90:
        a_count += 1
    elif grade >= 80:
        b_count += 1
    elif grade >= 70:
        c_count += 1
    elif grade >= 60:
        d_count += 1
    else:
        f_count += 1

print(f"A: {a_count} students")
print(f"B: {b_count} students")
print(f"C: {c_count} students")
print(f"D: {d_count} students")
print(f"F: {f_count} students")
```

### Example 2: Finding Outliers

```python
data = [10, 12, 9, 11, 45, 10, 8, 11, 100, 12]   # 45 and 100 are outliers

mean = sum(data) / len(data)
threshold = 2 * mean   # simple outlier rule

print(f"Mean: {mean:.2f}")
print(f"Outlier threshold: {threshold:.2f}")
print()

outliers = []
clean_data = []

for value in data:
    if value > threshold:
        outliers.append(value)
        print(f"  Outlier detected: {value}")
    else:
        clean_data.append(value)

print(f"\nOriginal: {data}")
print(f"Clean:    {clean_data}")
print(f"Outliers: {outliers}")
```

### Example 3: FizzBuzz (Classic Interview Problem)

```python
# Print 1-30, but:
# - Print "Fizz" for multiples of 3
# - Print "Buzz" for multiples of 5
# - Print "FizzBuzz" for multiples of both

for i in range(1, 31):
    if i % 15 == 0:           # 15 = 3 × 5
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

---

## 📊 Control Flow in Data Science — Why It Matters

| Scenario | Control Flow Used |
|----------|------------------|
| Categorize continuous variable | `if/elif/else` |
| Apply different logic to different data types | `if isinstance(...)` |
| Process each row in a dataset | `for row in dataframe.iterrows()` |
| Retry failed API calls | `while` with counter |
| Skip rows with missing values | `if pd.isna(value): continue` |
| Stop processing when quota reached | `break` |
| Create derived columns | List comprehension |

---

## ✅ Key Takeaways

- `if/elif/else` lets your program make **decisions** based on data
- `for` loops iterate over any **sequence** (lists, strings, ranges)
- `while` loops repeat until a **condition becomes False** — always have an exit strategy!
- `break` exits the loop; `continue` skips to the next iteration; `pass` does nothing
- **List comprehensions** are the Pythonic way to build lists from loops
- `enumerate()` and `zip()` make `for` loops much more powerful

---

## 🔗 What's Next?

➡️ [[04-functions]] — Package reusable logic into functions