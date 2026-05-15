# Practice Problems — Python Basics

Reading about code is not the same as writing it. These problems are designed to force you to think through the mechanics yourself. Work through each problem before reading the solution. If you get stuck, re-read the relevant note — the answer is always there.

> [!tip]
> Run every solution in a Python environment to confirm the output matches. If your answer is different from the expected output, the discrepancy is worth investigating before moving on.

---

## Level 1 — Warm-Up

These problems test whether you have absorbed the fundamentals. Each should take 5–10 minutes.

---

### Problem 1: Temperature Converter

Write two functions: `celsius_to_fahrenheit(c)` and `fahrenheit_to_celsius(f)`.

Formula: `F = C × 9/5 + 32`

Also write a `convert_temperature(value, from_unit)` dispatcher that accepts `"C"` or `"F"` and returns the converted value along with its unit.

Expected output:
```
0°C → 32.0°F
100°C → 212.0°F
-40°C → -40.0°F
98.6°F → 37.0°C
0°F → -17.8°C
```

```python
def celsius_to_fahrenheit(c):
    return c * 9 / 5 + 32

def fahrenheit_to_celsius(f):
    return (f - 32) * 5 / 9

def convert_temperature(value, from_unit):
    """Convert temperature and return (result, to_unit) tuple."""
    if from_unit.upper() == "C":
        return celsius_to_fahrenheit(value), "F"
    elif from_unit.upper() == "F":
        return fahrenheit_to_celsius(value), "C"
    else:
        raise ValueError(f"Unknown unit: {from_unit}. Use 'C' or 'F'.")

test_cases = [(0, "C"), (100, "C"), (-40, "C"), (98.6, "F"), (0, "F")]
for value, unit in test_cases:
    result, to_unit = convert_temperature(value, unit)
    print(f"{value}°{unit} → {round(result, 1)}°{to_unit}")
```

---

### Problem 2: String Statistics

Write a function `analyze_string(text)` that returns a dictionary with:
- `word_count`: number of words
- `char_count`: total characters (excluding spaces)
- `sentence_count`: number of sentences (count `.`, `!`, `?`)
- `avg_word_length`: average word length (rounded to 2 decimal places)
- `longest_word`: the longest word in the text (lowercase)

Expected output for `"Python is great! Data science is powerful. Use Python."`:
```
{'word_count': 9, 'char_count': 40, 'sentence_count': 3, 'avg_word_length': 4.44, 'longest_word': 'science'}
```

```python
def analyze_string(text):
    """Return a statistical summary of the given text."""
    words = text.split()
    word_lengths = [len(w.strip(".,!?")) for w in words]
    sentence_terminators = sum(1 for ch in text if ch in ".!?")

    return {
        "word_count": len(words),
        "char_count": len(text.replace(" ", "")),
        "sentence_count": sentence_terminators,
        "avg_word_length": round(sum(word_lengths) / len(word_lengths), 2) if words else 0,
        "longest_word": max(words, key=lambda w: len(w.strip(".,!?"))).strip(".,!?").lower(),
    }

sample = "Python is great! Data science is powerful. Use Python."
stats = analyze_string(sample)
print(stats)
```

---

### Problem 3: Palindrome Checker

Write a function `is_palindrome(text)` that:
- Ignores case
- Ignores spaces and punctuation
- Returns `True` if the cleaned string reads the same forwards and backwards

Expected output:
```
racecar → True
hello → False
A man a plan a canal Panama → True
No lemon no melon → True
Was it a car or a cat I saw → True
Data science → False
```

```python
def is_palindrome(text):
    """Return True if text is a palindrome (case and punctuation insensitive)."""
    # Keep only alphanumeric characters, convert to lowercase
    cleaned = "".join(ch.lower() for ch in text if ch.isalnum())
    return cleaned == cleaned[::-1]

test_phrases = [
    "racecar",
    "hello",
    "A man a plan a canal Panama",
    "No lemon no melon",
    "Was it a car or a cat I saw",
    "Data science",
]

for phrase in test_phrases:
    result = is_palindrome(phrase)
    print(f"{phrase} → {result}")
```

---

## Level 2 — Core Skills

These problems require combining multiple concepts. Expect 15–25 minutes each.

---

### Problem 4: Data Validator

You are building a data ingestion pipeline. Write a function `validate_record(record)` that checks a customer record dict against these rules:

- `name`: required, must be a non-empty string
- `age`: required, must be an integer between 0 and 120
- `email`: required, must contain exactly one `@` and at least one `.` after the `@`
- `salary`: optional, but if present must be a positive number

Return a dict with:
- `is_valid`: `True` only if all required fields pass
- `errors`: list of error message strings (empty if valid)
- `cleaned`: a dict with name title-cased, age as int, email lowercased

Expected output for a bad record:
```
{'is_valid': False, 'errors': ['age must be between 0 and 120', 'email format invalid'], 'cleaned': {...}}
```

```python
def validate_record(record):
    """Validate a customer record and return validation results."""
    errors = []
    cleaned = {}

    # Validate name
    name = record.get("name", "")
    if not isinstance(name, str) or not name.strip():
        errors.append("name is required and must be a non-empty string")
        cleaned["name"] = None
    else:
        cleaned["name"] = name.strip().title()

    # Validate age
    raw_age = record.get("age")
    try:
        age = int(raw_age)
        if not (0 <= age <= 120):
            errors.append("age must be between 0 and 120")
            cleaned["age"] = None
        else:
            cleaned["age"] = age
    except (ValueError, TypeError):
        errors.append(f"age must be a number, got {repr(raw_age)}")
        cleaned["age"] = None

    # Validate email
    email = record.get("email", "")
    if not isinstance(email, str) or email.count("@") != 1:
        errors.append("email format invalid")
        cleaned["email"] = None
    else:
        local, domain = email.split("@")
        if not local or "." not in domain:
            errors.append("email format invalid")
            cleaned["email"] = None
        else:
            cleaned["email"] = email.lower().strip()

    # Validate salary (optional)
    if "salary" in record:
        try:
            salary = float(record["salary"])
            if salary < 0:
                errors.append("salary must be positive")
                cleaned["salary"] = None
            else:
                cleaned["salary"] = salary
        except (ValueError, TypeError):
            errors.append(f"salary must be a number, got {repr(record['salary'])}")
            cleaned["salary"] = None

    return {
        "is_valid": len(errors) == 0,
        "errors": errors,
        "cleaned": cleaned,
    }

test_records = [
    {"name": "alice sharma", "age": "30", "email": "Alice@Example.com", "salary": "75000"},
    {"name": "", "age": "150", "email": "not-an-email", "salary": "-500"},
    {"name": "Bob", "age": "25", "email": "bob@company.co.in"},
    {"name": None, "age": "abc", "email": "bad@"},
]

for i, record in enumerate(test_records, 1):
    result = validate_record(record)
    status = "VALID" if result["is_valid"] else "INVALID"
    print(f"\nRecord {i}: {status}")
    if result["errors"]:
        for err in result["errors"]:
            print(f"  - {err}")
    print(f"  Cleaned: {result['cleaned']}")
```

---

### Problem 5: Word Frequency Analysis

Given a block of text, write a function `top_words(text, n=10, min_length=3)` that:
- Converts to lowercase
- Removes all punctuation
- Splits into words
- Filters out words shorter than `min_length`
- Returns the top `n` words by frequency as a list of `(word, count)` tuples

Expected output (for a reasonable sample text):
```
Top 5 words (min length 4):
  data       : 8
  science    : 5
  python     : 4
  model      : 4
  features   : 3
```

```python
def top_words(text, n=10, min_length=3):
    """Return the n most frequent words of at least min_length characters."""
    import string

    # Clean and split
    cleaned = text.lower().translate(str.maketrans("", "", string.punctuation))
    words = cleaned.split()

    # Count frequencies
    freq = {}
    for word in words:
        if len(word) >= min_length:
            freq[word] = freq.get(word, 0) + 1

    # Sort by count descending, then alphabetically for ties
    ranked = sorted(freq.items(), key=lambda x: (-x[1], x[0]))
    return ranked[:n]

sample_text = """
Data science is an interdisciplinary field that uses scientific methods,
processes, algorithms and systems to extract knowledge and insights from data.
Data science is related to data mining, machine learning and big data.
Python is the most popular programming language for data science and machine learning.
Data scientists use Python to build models that can analyze data and make predictions.
The field of data science draws from statistics, computer science and domain knowledge.
Machine learning models learn from data patterns to make intelligent predictions.
"""

results = top_words(sample_text, n=8, min_length=4)
print(f"Top {len(results)} words (min length 4):")
for word, count in results:
    bar = "#" * count
    print(f"  {word:<12}: {bar} ({count})")
```

---

### Problem 6: Running Statistics

Write a class `RunningStats` that computes statistics incrementally — one data point at a time — without storing all values. This is important for large datasets that do not fit in memory.

Implement:
- `add(value)`: add a new data point
- `mean`: property returning current mean
- `variance`: property returning current variance (population variance)
- `count`: property returning number of values added
- `__repr__`: show current stats

This uses Welford's online algorithm for numerically stable variance.

```python
class RunningStats:
    """
    Compute mean and variance incrementally using Welford's algorithm.
    Numerically stable — does not accumulate floating point error.
    """

    def __init__(self):
        self._count = 0
        self._mean = 0.0
        self._M2 = 0.0    # sum of squared deviations from mean

    def add(self, value):
        """Update statistics with a new data point."""
        self._count += 1
        delta = value - self._mean
        self._mean += delta / self._count
        delta2 = value - self._mean
        self._M2 += delta * delta2

    @property
    def count(self):
        return self._count

    @property
    def mean(self):
        return self._mean if self._count > 0 else 0.0

    @property
    def variance(self):
        return self._M2 / self._count if self._count > 0 else 0.0

    @property
    def std(self):
        return self.variance ** 0.5

    def __repr__(self):
        return (f"RunningStats(n={self.count}, mean={self.mean:.4f}, "
                f"std={self.std:.4f})")

# Test with a known dataset
stats = RunningStats()
sensor_readings = [10.2, 10.5, 10.1, 10.8, 10.3, 9.9, 10.6, 10.4, 10.7, 10.2]

for reading in sensor_readings:
    stats.add(reading)
    print(f"  Added {reading:.1f} → {stats}")

# Verify against direct calculation
import math
mean_direct = sum(sensor_readings) / len(sensor_readings)
var_direct = sum((x - mean_direct) ** 2 for x in sensor_readings) / len(sensor_readings)
print(f"\nDirect calculation: mean={mean_direct:.4f}, std={math.sqrt(var_direct):.4f}")
print(f"Running stats:      mean={stats.mean:.4f}, std={stats.std:.4f}")
```

---

## Level 3 — Data Science Focus

These problems are realistic data science tasks. Expect 30–45 minutes each.

---

### Problem 7: CSV Parser and Summarizer

Without using Pandas or the `csv` module, write a pure-Python CSV parser and summarizer.

Write:
- `parse_csv(lines, delimiter=",")` — parses lines into list of dicts using first row as headers
- `summarize_numeric_column(records, column)` — returns min, max, mean, median for a numeric column
- `group_by(records, column)` — groups records by a column value, returns a dict of lists

```python
def parse_csv(lines, delimiter=","):
    """Parse CSV lines (list of strings) into a list of dicts."""
    if not lines:
        return []
    headers = [h.strip() for h in lines[0].split(delimiter)]
    records = []
    for line in lines[1:]:
        if not line.strip():
            continue    # skip blank lines
        values = [v.strip() for v in line.split(delimiter)]
        # Pad or truncate to match header count
        while len(values) < len(headers):
            values.append("")
        record = dict(zip(headers, values[:len(headers)]))
        records.append(record)
    return records

def summarize_numeric_column(records, column):
    """Return summary statistics for a numeric column."""
    raw_values = [r.get(column) for r in records]
    numeric_values = []
    invalid_count = 0

    for v in raw_values:
        try:
            numeric_values.append(float(v))
        except (ValueError, TypeError):
            invalid_count += 1

    if not numeric_values:
        return {"error": f"No valid numeric values in column '{column}'"}

    numeric_values.sort()
    n = len(numeric_values)
    mean = sum(numeric_values) / n
    mid = n // 2
    median = numeric_values[mid] if n % 2 != 0 else (numeric_values[mid - 1] + numeric_values[mid]) / 2

    return {
        "column": column,
        "count": n,
        "invalid": invalid_count,
        "min": numeric_values[0],
        "max": numeric_values[-1],
        "mean": round(mean, 2),
        "median": round(median, 2),
    }

def group_by(records, column):
    """Group records by the value of a given column."""
    groups = {}
    for record in records:
        key = record.get(column, "MISSING")
        if key not in groups:
            groups[key] = []
        groups[key].append(record)
    return groups

# Test dataset
csv_data = """name,age,department,salary
Alice Sharma,32,Engineering,95000
Bob Chen,28,Marketing,62000
Charlie Patel,45,Engineering,120000
Diana Williams,31,Data Science,88000
Eve Johnson,27,Marketing,58000
Frank Kumar,38,Data Science,105000
Grace Tan,29,Engineering,82000
Henry Park,52,Management,145000
"""

lines = csv_data.strip().split("\n")
records = parse_csv(lines)

print(f"Parsed {len(records)} records\n")

# Salary statistics
salary_stats = summarize_numeric_column(records, "salary")
print("Salary Summary:")
for key, val in salary_stats.items():
    print(f"  {key:<10}: {val}")

# Group by department
print("\nBy Department:")
dept_groups = group_by(records, "department")
for dept, dept_records in sorted(dept_groups.items()):
    salaries = [float(r["salary"]) for r in dept_records]
    avg_salary = sum(salaries) / len(salaries)
    names = [r["name"].split()[0] for r in dept_records]
    print(f"  {dept:<15} n={len(dept_records)}  avg_salary={avg_salary:,.0f}  members={', '.join(names)}")
```

---

### Problem 8: Moving Average and Anomaly Detection

Stock price data (or any time series) often contains noise. A moving average smooths it out, and comparing the current value to the moving average can flag anomalies.

Write:
- `moving_average(values, window)` — computes the simple moving average with the given window
- `detect_anomalies(values, window, threshold_std=2.0)` — returns indices and values where the point deviates more than `threshold_std` standard deviations from the local moving average

```python
def moving_average(values, window):
    """
    Compute the simple moving average with the given window size.

    Returns a list of length len(values) - window + 1.
    The i-th result is the average of values[i : i + window].
    """
    if window > len(values):
        raise ValueError(f"Window {window} is larger than data length {len(values)}")
    return [
        sum(values[i : i + window]) / window
        for i in range(len(values) - window + 1)
    ]

def detect_anomalies(values, window, threshold_std=2.0):
    """
    Flag values that deviate more than threshold_std standard deviations
    from the corresponding moving average.

    Returns a list of (index, value, local_mean, deviation_in_std) tuples.
    """
    if window >= len(values):
        return []

    averages = moving_average(values, window)
    anomalies = []

    # Compute the standard deviation of the moving average residuals
    residuals = []
    for i, avg in enumerate(averages):
        center_idx = i + window // 2    # align window center to original index
        if center_idx < len(values):
            residuals.append(abs(values[center_idx] - avg))

    if not residuals:
        return []

    mean_residual = sum(residuals) / len(residuals)
    std_residual = (sum((r - mean_residual) ** 2 for r in residuals) / len(residuals)) ** 0.5

    if std_residual == 0:
        return []

    for i, avg in enumerate(averages):
        center_idx = i + window // 2
        if center_idx < len(values):
            deviation = abs(values[center_idx] - avg)
            std_deviations = deviation / std_residual
            if std_deviations > threshold_std:
                anomalies.append({
                    "index": center_idx,
                    "value": values[center_idx],
                    "local_mean": round(avg, 2),
                    "std_deviations": round(std_deviations, 2),
                })

    return anomalies

# Simulated stock prices with two anomalies injected
stock_prices = [
    100, 102, 101, 103, 102, 104, 103, 105, 104, 106,
    150,    # anomaly: spike
    105, 104, 106, 105, 107, 106, 108, 107, 109,
    60,     # anomaly: crash
    108, 109, 110, 111, 110,
]

window_size = 5
ma = moving_average(stock_prices, window_size)
print(f"Moving averages ({window_size}-period):")
for i, avg in enumerate(ma[:10]):
    print(f"  Window starting at {i}: {avg:.1f}")

print(f"\nTotal prices: {len(stock_prices)}, Moving averages computed: {len(ma)}")

anomalies = detect_anomalies(stock_prices, window=5, threshold_std=2.0)
print(f"\nAnomalies detected: {len(anomalies)}")
for a in anomalies:
    print(f"  Index {a['index']}: value={a['value']}, local_mean={a['local_mean']}, "
          f"deviation={a['std_deviations']} std")
```

---

## Challenge Problems

These are stretch problems for students who want an extra challenge. They require combining everything covered in Day 01.

---

### Challenge 1: Two Sum (Interview Classic)

Given a list of integers and a target, find the **two indices** whose values sum to the target. Each input has exactly one solution, and you may not use the same element twice.

Solve it in **O(n) time** using a hash map.

```python
def two_sum(numbers, target):
    """
    Return indices [i, j] such that numbers[i] + numbers[j] == target.
    Uses a hash map for O(n) time complexity.

    Args:
        numbers: list of integers
        target: int

    Returns:
        list of two indices, or [] if no solution exists
    """
    seen = {}    # value → index

    for i, num in enumerate(numbers):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i

    return []

# Tests
test_cases = [
    ([2, 7, 11, 15], 9),     # Expected: [0, 1]
    ([3, 2, 4], 6),           # Expected: [1, 2]
    ([3, 3], 6),              # Expected: [0, 1]
    ([1, 5, 3, 7, 2], 9),    # Expected: [1, 3]
]

for numbers, target in test_cases:
    result = two_sum(numbers, target)
    if result:
        i, j = result
        print(f"numbers={numbers}, target={target} → [{i}, {j}]  "
              f"(values: {numbers[i]} + {numbers[j]} = {numbers[i] + numbers[j]})")
    else:
        print(f"numbers={numbers}, target={target} → No solution")
```

---

### Challenge 2: Flatten and Summarize Nested JSON

Real-world API data often comes as deeply nested JSON. Write a function `flatten_dict(d, sep=".")` that recursively flattens a nested dictionary using dot notation for keys, then a `summarize_flat(flat_dict)` that categorizes every leaf value by its type.

```python
def flatten_dict(d, parent_key="", sep="."):
    """
    Flatten a nested dictionary with dot-notation keys.

    Example:
        {"a": {"b": 1, "c": {"d": 2}}} → {"a.b": 1, "a.c.d": 2}
    """
    items = []
    for key, value in d.items():
        new_key = f"{parent_key}{sep}{key}" if parent_key else key
        if isinstance(value, dict):
            items.extend(flatten_dict(value, new_key, sep).items())
        elif isinstance(value, list):
            for idx, item in enumerate(value):
                list_key = f"{new_key}[{idx}]"
                if isinstance(item, dict):
                    items.extend(flatten_dict(item, list_key, sep).items())
                else:
                    items.append((list_key, item))
        else:
            items.append((new_key, value))
    return dict(items)

def summarize_flat(flat_dict):
    """Summarize a flat dict by grouping keys by their value type."""
    summary = {"str": [], "int": [], "float": [], "bool": [], "none": [], "other": []}
    for key, value in flat_dict.items():
        if isinstance(value, bool):
            summary["bool"].append(key)
        elif isinstance(value, int):
            summary["int"].append(key)
        elif isinstance(value, float):
            summary["float"].append(key)
        elif isinstance(value, str):
            summary["str"].append(key)
        elif value is None:
            summary["none"].append(key)
        else:
            summary["other"].append(key)
    return {k: v for k, v in summary.items() if v}    # drop empty categories

# Test with a realistic API response
api_payload = {
    "status": "success",
    "data": {
        "user": {
            "id": 10042,
            "name": "Alice Sharma",
            "is_premium": True,
            "preferences": {
                "language": "en",
                "currency": "INR",
                "notifications": True,
            },
        },
        "metrics": {
            "sessions": 47,
            "total_spend": 12450.75,
            "churn_probability": 0.08,
            "last_active": "2024-01-15",
            "inactive_since": None,
        },
        "recent_purchases": [
            {"item": "laptop", "amount": 85000},
            {"item": "headphones", "amount": 3500},
        ],
    },
}

flat = flatten_dict(api_payload)
print("Flattened keys:")
for key, value in flat.items():
    print(f"  {key}: {repr(value)}")

print("\nType summary:")
type_summary = summarize_flat(flat)
for dtype, keys in type_summary.items():
    print(f"  {dtype}: {', '.join(keys)}")
```

---

[[05-lists-tuples-dictionaries]] | [[07-interview-questions]]
