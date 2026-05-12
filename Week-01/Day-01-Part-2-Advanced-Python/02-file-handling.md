# 📁 02 – File Handling

> **Prerequisites:** [[01-oop-basics]]  
> **Time to read:** ~25 minutes

---

## 🧠 Why File Handling?

Every Data Science project reads data from somewhere — CSVs, JSON files, logs, text files. Understanding Python's file I/O gives you control before Pandas even enters the picture.

---

## 📖 Reading Text Files

### The Basic Pattern

```python
# Always use 'with' — it automatically closes the file
with open("data.txt", "r") as f:
    content = f.read()        # read entire file as one string
    print(content)
```

### `open()` Modes

| Mode | Meaning |
|------|---------|
| `"r"` | Read (default) — file must exist |
| `"w"` | Write — creates new or overwrites |
| `"a"` | Append — adds to end of file |
| `"x"` | Exclusive create — fails if file exists |
| `"b"` | Binary mode (combine: `"rb"`, `"wb"`) |
| `"+"` | Read + Write (combine: `"r+"`) |

### Reading Methods

```python
with open("data.txt", "r", encoding="utf-8") as f:
    # Read all at once
    content = f.read()

with open("data.txt", "r", encoding="utf-8") as f:
    # Read all lines into a list
    lines = f.readlines()         # [line1\n, line2\n, ...]

with open("data.txt", "r", encoding="utf-8") as f:
    # Read one line at a time — memory efficient for large files!
    for line in f:
        print(line.strip())       # strip() removes the \n

with open("data.txt", "r") as f:
    # Read first line only
    first_line = f.readline()
```

---

## ✏️ Writing Text Files

```python
# Write (creates file, overwrites if exists)
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!\n")
    f.write("Second line.\n")

# Write multiple lines at once
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)

# Append (adds to existing file)
with open("log.txt", "a") as f:
    f.write("New log entry\n")

# Using print() to write to file
with open("output.txt", "w") as f:
    print("Hello from print!", file=f)
    print(f"Current time: 2025", file=f)
```

---

## 📊 Working with CSV Files

### Using `csv` module (built-in)

```python
import csv

# Reading CSV
with open("students.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)    # reads rows as dicts using header row
    for row in reader:
        print(row)
        # {'name': 'Alice', 'age': '30', 'score': '85'}

# Read into a list
with open("students.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    data = list(reader)

# Writing CSV
headers = ["name", "age", "score"]
records = [
    {"name": "Alice", "age": 30, "score": 85},
    {"name": "Bob", "age": 25, "score": 92},
]

with open("output.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=headers)
    writer.writeheader()          # write header row
    writer.writerows(records)     # write all data rows
```

### Create a CSV from scratch for practice

```python
import csv

# Create sample data
sample_data = [
    {"name": "Alice", "age": 30, "salary": 80000, "city": "Mumbai"},
    {"name": "Bob", "age": 25, "salary": 55000, "city": "Delhi"},
    {"name": "Charlie", "age": 35, "salary": 120000, "city": "Bangalore"},
]

with open("employees.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age", "salary", "city"])
    writer.writeheader()
    writer.writerows(sample_data)

print("CSV created!")

# Read it back
with open("employees.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['name']}: ₹{int(row['salary']):,}")
```

---

## 🔧 Working with JSON Files

JSON is the most common format for APIs and configuration files.

```python
import json

# Writing JSON
data = {
    "model": "LinearRegression",
    "version": "1.0",
    "hyperparameters": {
        "learning_rate": 0.01,
        "epochs": 100
    },
    "features": ["age", "salary", "experience"],
    "accuracy": 0.92
}

with open("model_config.json", "w") as f:
    json.dump(data, f, indent=4)   # indent=4 for pretty printing

# Reading JSON
with open("model_config.json", "r") as f:
    loaded = json.load(f)

print(loaded["model"])             # LinearRegression
print(loaded["hyperparameters"])   # {'learning_rate': 0.01, 'epochs': 100}
print(loaded["features"][0])       # age

# JSON ↔ String (no file)
json_string = json.dumps(data, indent=2)   # dict → string
print(json_string[:50])

parsed = json.loads(json_string)           # string → dict
```

---

## 🛣️ Working with File Paths (pathlib)

```python
from pathlib import Path

# Create path objects
current = Path(".")
home = Path.home()
data_dir = Path("data")
file = Path("data/students.csv")

# Build paths safely (works on Windows AND Mac/Linux)
report = Path("reports") / "2025" / "january" / "summary.txt"
print(report)   # reports/2025/january/summary.txt

# File info
p = Path("students.csv")
print(p.exists())       # True/False
print(p.name)           # "students.csv"
print(p.stem)           # "students"
print(p.suffix)         # ".csv"
print(p.parent)         # "." (current dir)

# Create directories
output_dir = Path("output")
output_dir.mkdir(exist_ok=True)              # create, don't error if exists
output_dir.mkdir(parents=True, exist_ok=True) # create all parent dirs too

# List files
data_path = Path(".")
csv_files = list(data_path.glob("*.csv"))   # all .csv in current dir
all_md = list(data_path.rglob("*.md"))      # recursive — all .md files

# Read/write with pathlib
file = Path("notes.txt")
file.write_text("Hello, World!", encoding="utf-8")
content = file.read_text(encoding="utf-8")
```

---

## 🧪 Data Science File Pattern

```python
import csv
import json
from pathlib import Path

def load_csv(filepath):
    """Load a CSV file into a list of dicts."""
    with open(filepath, "r", newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))

def save_csv(data, filepath, fieldnames=None):
    """Save a list of dicts to CSV."""
    if not data:
        return
    if fieldnames is None:
        fieldnames = list(data[0].keys())

    Path(filepath).parent.mkdir(parents=True, exist_ok=True)

    with open(filepath, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)

def save_json(data, filepath):
    """Save data to JSON with pretty printing."""
    Path(filepath).parent.mkdir(parents=True, exist_ok=True)
    with open(filepath, "w") as f:
        json.dump(data, f, indent=2)

def load_json(filepath):
    """Load JSON from file."""
    with open(filepath, "r") as f:
        return json.load(f)

# Example pipeline
records = load_csv("input.csv")
processed = [{**r, "salary": float(r["salary"])} for r in records]
save_csv(processed, "output/clean_data.csv")
save_json({"count": len(processed), "fields": list(processed[0].keys())}, "output/summary.json")
```

---

## ✅ Key Takeaways

- Always use `with open(...)` — it closes the file automatically
- Always specify `encoding="utf-8"` for text files on multi-platform projects
- `csv.DictReader` / `DictWriter` make CSV work easy and readable
- `json.dump()` / `json.load()` for JSON files; `json.dumps()` / `json.loads()` for strings
- Use `pathlib.Path` — it handles paths correctly on all operating systems
- For large files, iterate line-by-line instead of reading all at once

---

## 🔗 What's Next?

➡️ [[03-modules-and-packages]] — Import and organize code into reusable modules