# File Handling

Before Pandas, before SQL, there are files. A large share of real data science work starts with raw CSVs, JSON configs, log files, or text dumps that someone emailed you. Knowing how to read and write files reliably — without resource leaks or encoding surprises — is the foundation everything else builds on.

**Prerequisites:** [[01-oop-basics]]

## Learning Objectives

- Use `with open()` and explain precisely why it matters
- Know the correct mode string for every file operation
- Read and write text, CSV, and JSON files with proper encoding handling
- Use `pathlib.Path` to build OS-independent file paths
- Recognize and avoid the three most common file-handling mistakes

---

## Why `with open()` — Not Just `open()`

This is the single most important habit in Python file handling.

```python
# Dangerous — if anything raises an exception between open() and close(),
# the file handle is never closed. On long-running processes, you eventually
# run out of file descriptors.
f = open("data.txt", "r")
content = f.read()
f.close()

# Correct — the context manager guarantees the file is closed,
# even if an exception occurs inside the block.
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()
# f is now closed — no matter what happened inside
```

> [!warning]
> In production code I have seen file descriptor leaks cause servers to stop accepting connections after several hours of uptime. The fix was replacing every bare `open()` + `close()` with `with open()`. Always use the context manager.

---

## `open()` Modes

| Mode | What it does | File must exist? |
|------|-------------|------------------|
| `"r"` | Read text (default) | Yes — raises `FileNotFoundError` if missing |
| `"w"` | Write text — creates or **overwrites** | No — creates it |
| `"a"` | Append text — adds to end | No — creates it |
| `"x"` | Exclusive create — fails if file exists | No — raises `FileExistsError` if present |
| `"r+"` | Read + write, file must exist | Yes |
| `"rb"` | Read binary | Yes |
| `"wb"` | Write binary | No |

> [!warning]
> Mode `"w"` silently overwrites the entire file. There is no undo. If you only want to add data, use `"a"`.

---

## Reading Text Files

```python
# Read the entire file as a single string
with open("report.txt", "r", encoding="utf-8") as f:
    full_text = f.read()

print(type(full_text))   # Output: <class 'str'>
print(len(full_text))    # Output: number of characters


# Read all lines into a list — each line includes the trailing \n
with open("report.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()

# strip() removes the trailing newline
for line in lines:
    print(line.strip())


# Iterate line by line — best for large files, minimal memory use
with open("report.txt", "r", encoding="utf-8") as f:
    for line in f:
        clean_line = line.strip()
        if clean_line:  # skip blank lines
            print(clean_line)


# Read only the first line
with open("report.txt", "r", encoding="utf-8") as f:
    header = f.readline().strip()
    print(header)
```

> [!tip]
> For files larger than a few hundred MB, always iterate line by line. `f.read()` loads the entire file into memory. Iterating the file object directly is memory-efficient — Python reads one chunk at a time.

---

## Writing Text Files

```python
# Write — creates or overwrites
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Line one\n")
    f.write("Line two\n")


# writelines() writes a list — note: it does NOT add newlines for you
lines = ["First line\n", "Second line\n", "Third line\n"]
with open("output.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)


# Append — adds to the end, never overwrites
with open("pipeline.log", "a", encoding="utf-8") as f:
    f.write("2025-01-15 14:30 — training complete\n")


# print() works too — file= parameter redirects its output
with open("results.txt", "w", encoding="utf-8") as f:
    print("Accuracy: 0.94", file=f)
    print("F1 Score: 0.91", file=f)
```

> [!info]
> Always specify `encoding="utf-8"` explicitly. The default encoding is platform-dependent: on Windows it is often `cp1252`, which will silently corrupt non-ASCII characters (accented names, currency symbols, etc.) when you share the file with a Linux or Mac system.

---

## Working with CSV Files

CSV is the most common raw data format you will encounter. Python's built-in `csv` module handles edge cases (quoted fields with commas, embedded newlines) that string splitting misses.

### Reading CSV

```python
import csv

# csv.DictReader maps each row to a dict using the header row as keys
with open("employees.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
        # Output: {'name': 'Alice', 'department': 'Engineering', 'salary': '95000'}


# Load all rows at once
with open("employees.csv", "r", newline="", encoding="utf-8") as f:
    records = list(csv.DictReader(f))

print(f"Loaded {len(records)} records")
print(records[0])
```

> [!warning]
> `csv.DictReader` reads every value as a **string**. `salary` will be `"95000"`, not `95000`. You must convert types yourself: `int(row["salary"])` or `float(row["price"])`.

### Writing CSV

```python
import csv

employee_records = [
    {"name": "Alice", "department": "Engineering", "salary": 95000},
    {"name": "Bob", "department": "Marketing", "salary": 72000},
    {"name": "Carol", "department": "Engineering", "salary": 88000},
]

with open("employees.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "department", "salary"])
    writer.writeheader()      # writes the column name row
    writer.writerows(employee_records)

print("File written.")
```

> [!info]
> The `newline=""` argument is required on Windows when using the `csv` module. Without it, the writer inserts an extra blank line between every row because Windows line endings interact with the csv writer's own line ending logic.

### Creating and Reading Back a CSV

```python
import csv

# Write a CSV from scratch
sales_data = [
    {"product": "Laptop", "units_sold": 12, "unit_price": 75000},
    {"product": "Monitor", "units_sold": 8, "unit_price": 15000},
    {"product": "Keyboard", "units_sold": 25, "unit_price": 1200},
]

with open("sales_q1.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["product", "units_sold", "unit_price"])
    writer.writeheader()
    writer.writerows(sales_data)

# Read it back and compute revenue
total_revenue = 0
with open("sales_q1.csv", "r", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        revenue = int(row["units_sold"]) * float(row["unit_price"])
        total_revenue += revenue
        print(f"{row['product']}: {revenue:,.0f}")

print(f"\nTotal Q1 Revenue: {total_revenue:,.0f}")
# Output:
# Laptop: 900,000
# Monitor: 120,000
# Keyboard: 30,000
#
# Total Q1 Revenue: 1,050,000
```

---

## Working with JSON Files

JSON is universal for configs, API responses, and storing structured results.

```python
import json

# Writing JSON to a file
model_results = {
    "model": "RandomForestClassifier",
    "run_id": "2025-01-15-001",
    "hyperparameters": {"n_estimators": 200, "max_depth": 10, "min_samples_split": 5},
    "metrics": {"accuracy": 0.9412, "f1_score": 0.9187, "roc_auc": 0.9803},
    "feature_columns": ["age", "income", "tenure", "plan_type"],
}

with open("model_results.json", "w", encoding="utf-8") as f:
    json.dump(model_results, f, indent=2)  # indent=2 for readable output


# Reading JSON from a file
with open("model_results.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)

print(loaded["model"])                     # Output: RandomForestClassifier
print(loaded["metrics"]["accuracy"])       # Output: 0.9412
print(loaded["hyperparameters"])           # Output: {'n_estimators': 200, ...}


# JSON ↔ string (no file involved)
json_string = json.dumps(model_results, indent=2)   # dict → JSON string
print(type(json_string))                             # Output: <class 'str'>

parsed = json.loads(json_string)                    # JSON string → dict
print(parsed["run_id"])                              # Output: 2025-01-15-001
```

> [!warning]
> `json.dump()` writes to a file object. `json.dumps()` returns a string. `json.load()` reads from a file. `json.loads()` parses a string. The `s` stands for "string". Mixing these up is a very common source of `TypeError`.

---

## File Paths with `pathlib`

String-based paths (`"data/2025/january/report.csv"`) break on Windows vs Unix. `pathlib.Path` handles this automatically and provides useful methods.

```python
from pathlib import Path

# Build paths safely — the / operator joins path components
project_root = Path(".")
data_dir = project_root / "data"
report_path = data_dir / "2025" / "january" / "report.csv"

print(report_path)          # Output: data/2025/january/report.csv (or data\2025\... on Windows)


# Inspect a path
p = Path("data/employees.csv")
print(p.name)       # Output: employees.csv
print(p.stem)       # Output: employees
print(p.suffix)     # Output: .csv
print(p.parent)     # Output: data
print(p.exists())   # Output: True or False


# Create directories
output_dir = Path("output") / "reports" / "2025"
output_dir.mkdir(parents=True, exist_ok=True)
# parents=True: creates all intermediate directories
# exist_ok=True: does not raise if the directory already exists


# Find files
data_path = Path("data")
if data_path.exists():
    csv_files = list(data_path.glob("*.csv"))        # all CSVs in data/
    all_csvs = list(data_path.rglob("*.csv"))        # all CSVs recursively
    print(f"Found {len(all_csvs)} CSV files")


# Read and write files directly through Path
config_path = Path("config.json")
config_path.write_text('{"version": "1.0"}', encoding="utf-8")
content = config_path.read_text(encoding="utf-8")
print(content)  # Output: {"version": "1.0"}
```

> [!tip]
> `Path` objects work everywhere a string path works in Python — you can pass them directly to `open()`, `json.load()`, `csv.reader()`, and pandas functions. No need to call `str(path)`.

---

## Reusable File Utilities

This is the pattern I use in most projects — small, focused functions for loading and saving common formats:

```python
import csv
import json
from pathlib import Path


def load_csv(filepath: str | Path) -> list[dict]:
    """Load a CSV file and return a list of row dicts."""
    filepath = Path(filepath)
    if not filepath.exists():
        raise FileNotFoundError(f"CSV not found: {filepath}")
    with open(filepath, "r", newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))


def save_csv(records: list[dict], filepath: str | Path) -> None:
    """Save a list of dicts to CSV, creating parent dirs as needed."""
    if not records:
        raise ValueError("Cannot save empty records list")
    filepath = Path(filepath)
    filepath.parent.mkdir(parents=True, exist_ok=True)
    with open(filepath, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=list(records[0].keys()))
        writer.writeheader()
        writer.writerows(records)


def load_json(filepath: str | Path) -> dict | list:
    """Load a JSON file."""
    filepath = Path(filepath)
    if not filepath.exists():
        raise FileNotFoundError(f"JSON not found: {filepath}")
    with open(filepath, "r", encoding="utf-8") as f:
        return json.load(f)


def save_json(data: dict | list, filepath: str | Path) -> None:
    """Save data to a JSON file with pretty printing."""
    filepath = Path(filepath)
    filepath.parent.mkdir(parents=True, exist_ok=True)
    with open(filepath, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)


# Example usage
raw_records = load_csv("data/raw_employees.csv")

# Clean salary column
for record in raw_records:
    record["salary"] = float(record["salary"])

save_csv(raw_records, "output/clean_employees.csv")

summary = {
    "record_count": len(raw_records),
    "columns": list(raw_records[0].keys()) if raw_records else [],
}
save_json(summary, "output/summary.json")

print(f"Processed {len(raw_records)} records")
```

---

## Key Takeaways

> [!success]
> - Always use `with open()` — it closes the file even when exceptions occur
> - Always specify `encoding="utf-8"` for text files — the default is platform-dependent and unreliable
> - `csv.DictReader` reads every value as a string — convert types explicitly
> - Pass `newline=""` to `open()` when using the `csv` module on Windows
> - `json.dump` / `json.load` operate on files; `json.dumps` / `json.loads` operate on strings
> - Use `pathlib.Path` for all file path operations — it is cross-platform and has useful inspection methods
> - For large files, iterate line by line rather than loading everything at once

---

[[01-oop-basics|← OOP Basics]] | [[03-modules-and-packages|Modules & Packages →]]
