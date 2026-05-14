# 🏋️ 06 – Practice Problems

> **Prerequisites:** All of Day 01 Part 1  
> **Tip:** Try to solve each problem yourself before looking at the solution!

---

## 🟢 Level 1 — Warm Up

### Problem 1: Temperature Converter
Write a function `celsius_to_fahrenheit(c)` and `fahrenheit_to_celsius(f)`.
Formula: `F = C × 9/5 + 32`

```python
def celsius_to_fahrenheit(c):
    return c * 9/5 + 32

def fahrenheit_to_celsius(f):
    return (f - 32) * 5/9

# Test
print(celsius_to_fahrenheit(0))     # 32.0
print(celsius_to_fahrenheit(100))   # 212.0
print(fahrenheit_to_celsius(32))    # 0.0
print(fahrenheit_to_celsius(98.6))  # 37.0
```

### Problem 2: Even or Odd Counter
Given a list of numbers, count how many are even and how many are odd.

```python
def count_even_odd(numbers):
    even = sum(1 for n in numbers if n % 2 == 0)
    odd = len(numbers) - even
    return {"even": even, "odd": odd}

print(count_even_odd([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]))
# {'even': 5, 'odd': 5}
```

### Problem 3: Palindrome Checker
A palindrome reads the same forwards and backwards ("racecar", "level").

```python
def is_palindrome(text):
    cleaned = text.lower().replace(" ", "")
    return cleaned == cleaned[::-1]

print(is_palindrome("racecar"))      # True
print(is_palindrome("hello"))        # False
print(is_palindrome("A man a plan a canal Panama"))  # True
```

---

## 🟡 Level 2 — Core Skills

### Problem 4: FizzBuzz (Classic!)
Print numbers 1–100. For multiples of 3 print "Fizz", for multiples of 5 print "Buzz", for both print "FizzBuzz".

```python
for i in range(1, 101):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

### Problem 5: Find Duplicates
Given a list, return a list of elements that appear more than once.

```python
def find_duplicates(lst):
    freq = {}
    for item in lst:
        freq[item] = freq.get(item, 0) + 1
    return [k for k, v in freq.items() if v > 1]

print(find_duplicates([1, 2, 3, 2, 4, 3, 5]))  # [2, 3]
print(find_duplicates([1, 2, 3]))               # []
```

### Problem 6: Caesar Cipher
Encrypt a message by shifting each letter by `n` positions in the alphabet.

```python
def caesar_cipher(text, shift):
    result = []
    for char in text:
        if char.isalpha():
            base = ord('a') if char.islower() else ord('A')
            shifted = (ord(char) - base + shift) % 26 + base
            result.append(chr(shifted))
        else:
            result.append(char)
    return "".join(result)

print(caesar_cipher("Hello, World!", 3))   # Khoor, Zruog!
print(caesar_cipher("Khoor, Zruog!", -3))  # Hello, World!
```

---

## 🔴 Level 3 — Data Science Focus

### Problem 7: Student Grade Analyzer
Given a dict of student names and scores, compute and display statistics.

```python
def analyze_grades(scores_dict):
    scores = list(scores_dict.values())
    n = len(scores)
    mean = sum(scores) / n
    passing = {name: s for name, s in scores_dict.items() if s >= 60}

    print(f"Students: {n}")
    print(f"Average: {mean:.1f}")
    print(f"Highest: {max(scores)} ({max(scores_dict, key=scores_dict.get)})")
    print(f"Lowest:  {min(scores)} ({min(scores_dict, key=scores_dict.get)})")
    print(f"Passing: {len(passing)}/{n}")
    print(f"Pass Rate: {len(passing)/n*100:.1f}%")

scores = {
    "Alice": 85, "Bob": 92, "Charlie": 45,
    "Diana": 78, "Eve": 60, "Frank": 55
}
analyze_grades(scores)
```

### Problem 8: CSV Row Parser
Parse a list of CSV strings into a list of dicts.

```python
def parse_csv(lines, delimiter=","):
    if not lines:
        return []
    headers = [h.strip() for h in lines[0].split(delimiter)]
    records = []
    for line in lines[1:]:
        values = [v.strip() for v in line.split(delimiter)]
        record = dict(zip(headers, values))
        records.append(record)
    return records

csv_data = [
    "name, age, city",
    "Alice, 30, Mumbai",
    "Bob, 25, Delhi",
    "Charlie, 35, Bangalore"
]

records = parse_csv(csv_data)
for r in records:
    print(r)
```

### Problem 9: Moving Average
Compute the moving average (rolling mean) of a list with a given window size.

```python
def moving_average(data, window):
    """Compute moving average with given window size."""
    if window > len(data):
        return []
    return [
        sum(data[i:i+window]) / window
        for i in range(len(data) - window + 1)
    ]

prices = [10, 12, 11, 14, 13, 15, 16, 14, 17, 18]
ma3 = moving_average(prices, 3)
print([round(x, 2) for x in ma3])
# [11.0, 12.33, 12.67, 14.0, 14.67, 15.0, 15.67, 16.33]
```

---

## 🧠 Challenge Problems

### Challenge 1: Two Sum
Given a list and a target, find two indices whose values sum to the target.

```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))   # [0, 1]
print(two_sum([3, 2, 4], 6))        # [1, 2]
```

### Challenge 2: Flatten Nested Dict
Given a nested dictionary, flatten it with dot-notation keys.

```python
def flatten_dict(d, parent_key="", sep="."):
    items = []
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_dict(v, new_key, sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

nested = {
    "user": {
        "name": "Alice",
        "address": {
            "city": "Mumbai",
            "zip": "400001"
        }
    },
    "score": 95
}

print(flatten_dict(nested))
# {'user.name': 'Alice', 'user.address.city': 'Mumbai', 'user.address.zip': '400001', 'score': 95}
```
