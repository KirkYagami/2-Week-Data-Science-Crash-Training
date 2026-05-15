# Day 02 — Part 2: Pandas Basics

Pandas is the tool you will use more than any other in data science work. Every dataset you analyze, every model you prepare data for, every report you generate — it starts with a DataFrame. This session builds the muscle memory you need before any real analysis can happen.

## Learning Objectives

By the end of this session, you will be able to:

- Explain what a `Series` and `DataFrame` are and how they relate to each other
- Understand the Index as a first-class object, not just row numbers
- Create DataFrames from dictionaries and lists
- Load CSV and Excel files with full control over which rows, columns, and types get loaded
- Select columns and rows confidently using `[]`, `.loc`, and `.iloc`
- Filter rows using boolean indexing and `.query()`
- Sort data and retrieve top/bottom records
- Run a first-look analysis with `.describe()`, `.value_counts()`, `.info()`, and `.corr()`
- Detect and quantify missing values
- Save results back to disk correctly

---

## Today's Roadmap

```text
Pandas Basics
│
├── 01 → Series and DataFrames
│         Series | DataFrame | Index | dtypes | memory | column selection
│
├── 02 → Reading CSV and Excel Files
│         read_csv parameters | encoding | messy CSVs | read_excel | saving
│
├── 03 → Filtering and Sorting
│         boolean indexing | .loc vs .iloc | .query() | sort_values | nlargest
│
├── 04 → Basic Data Analysis
│         describe | info | value_counts | corr | groupby preview | missing values
│
├── 05 → Practice Exercises
│         three levels: warm-up, main, stretch
│
└── 06 → Interview Questions
          collapsible model answers for 12 questions
```

---

## Time Allocation

| # | Topic | Duration | Type |
|---|-------|----------|------|
| 01 | Series and DataFrames | 35 min | Lecture + Code |
| 02 | Reading CSV and Excel Files | 30 min | Demo |
| 03 | Filtering and Sorting | 35 min | Lecture + Practice |
| 04 | Basic Data Analysis | 30 min | Guided Analysis |
| 05 | Practice Exercises | 50 min | Hands-on |
| 06 | Interview Questions | Reference | Revision |

**Total: ~3 hours**

---

## Prerequisites

- Python basics (functions, lists, dictionaries, loops)
- NumPy fundamentals (arrays, dtypes, vectorized operations)
- Jupyter Notebook, VS Code, or another Python environment
- Pandas installed

---

## Setup Check

```python
import pandas as pd
import numpy as np

print(pd.__version__)
# Output: 2.x.x  (any 1.5+ works for this material)
```

If Pandas is missing:

```bash
pip install pandas openpyxl
```

`openpyxl` is required for reading and writing `.xlsx` files. Install it alongside pandas.

---

## Files in This Module

| File | Topic |
|------|-------|
| `01-series-and-dataframes.md` | Core objects, Index, dtypes, memory, column selection |
| `02-reading-csv-excel.md` | Loading and saving files with full parameter control |
| `03-filtering-and-sorting.md` | Boolean indexing, .loc vs .iloc, query, sort |
| `04-basic-data-analysis.md` | Summary statistics, missing values, groupby preview |
| `05-practice-exercises.md` | Warm-up, main, and stretch exercises |
| `06-interview-questions.md` | 12 interview questions with model answers |

---

## Study Strategy

1. Type every code example yourself. Do not copy-paste. The act of typing builds pattern recognition.
2. Add `print()` calls liberally. Intermediate results reveal how pandas thinks.
3. Always inspect a dataset before analyzing it — shape, dtypes, missing values, first rows.
4. Read error messages top to bottom. The useful part is often the last two lines.
5. Keep the NumPy notes nearby. Pandas is built on NumPy and borrows its logic for operations.

---

[[../Day-02-Part-1-NumPy-Fundamentals/07-cheat-sheet|Previous: NumPy Cheat Sheet]] | [[01-series-and-dataframes|Next: Series and DataFrames]]
