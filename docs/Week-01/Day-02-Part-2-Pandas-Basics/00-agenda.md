# 📅 Day 02 — Part 2: Pandas Basics
## Agenda & Learning Roadmap

> [!info] Overview
> Pandas is the main Python library for working with tabular data. If NumPy helps you think in arrays, Pandas helps you think in rows, columns, datasets, and real analysis workflows.

---

## 🗺️ Today's Roadmap

```text
Pandas Basics
│
├── 01 → Series and DataFrames
│         Series | DataFrame | columns | index | basic inspection
│
├── 02 → Reading CSV and Excel Files
│         read_csv | read_excel | file paths | export basics
│
├── 03 → Filtering and Sorting
│         boolean masks | loc | isin | query | sort_values
│
├── 04 → Basic Data Analysis
│         describe | value_counts | aggregations | simple insights
│
├── 05 → Practice Exercises
│         hands-on Pandas basics with solutions
│
└── 06 → Interview Questions
          common beginner Pandas interview questions
```

---

## ⏱️ Time Allocation

| # | Topic | Duration | Type |
|---|-------|----------|------|
| 01 | Series and DataFrames | 30 min | Lecture + Code |
| 02 | Reading CSV and Excel Files | 25 min | Demo |
| 03 | Filtering and Sorting | 30 min | Lecture + Practice |
| 04 | Basic Data Analysis | 30 min | Guided Analysis |
| 05 | Practice Exercises | 45 min | Hands-on |
| 06 | Interview Questions | Reference | Revision |

**Total: ~2.5 hours**

---

## 🎯 Learning Objectives

By the end of this session, you will be able to:

- [ ] Explain what a Pandas `Series` is
- [ ] Explain what a Pandas `DataFrame` is
- [ ] Create DataFrames from dictionaries and lists
- [ ] Read CSV and Excel files using Pandas
- [ ] Inspect rows, columns, shape, data types, and missing values
- [ ] Select columns and rows with `[]`, `.loc`, and `.iloc`
- [ ] Filter data using conditions
- [ ] Sort data by one or more columns
- [ ] Use basic analysis functions like `describe()`, `value_counts()`, and `mean()`
- [ ] Save cleaned or filtered data to a new file

---

## 🔧 Prerequisites

- [ ] Python basics
- [ ] NumPy fundamentals
- [ ] Jupyter Notebook, VS Code, or another Python editor
- [ ] Pandas installed

---

## 📦 Setup Check

```python
import pandas as pd

print(pd.__version__)
```

If Pandas is missing:

```bash
pip install pandas openpyxl
```

`openpyxl` is commonly used by Pandas for reading and writing Excel `.xlsx` files.

---

## 📚 Files In This Module

| File | Topic |
|------|-------|
| `01-series-and-dataframes.md` | Pandas objects and basic inspection |
| `02-reading-csv-excel.md` | Loading and saving tabular files |
| `03-filtering-and-sorting.md` | Selecting, filtering, and ordering data |
| `04-basic-data-analysis.md` | Quick summary and analysis methods |
| `05-practice-exercises.md` | Hands-on practice problems |
| `06-interview-questions.md` | Interview revision questions |

---

## 💡 Study Strategy

1. Type the examples yourself.
2. Print intermediate results often.
3. Always inspect a dataset before analyzing it.
4. Practice reading error messages carefully.
5. Keep the NumPy cheat sheet nearby because Pandas builds on NumPy concepts.

---

## 🔗 Previous and Next

| Previous | Next |
|----------|------|
| [[../Day-02-Part-1-NumPy-Fundamentals/07-cheat-sheet|NumPy Cheat Sheet]] | [[01-series-and-dataframes]] |

---

*Tags: #pandas #dataframes #series #csv #data-analysis*
