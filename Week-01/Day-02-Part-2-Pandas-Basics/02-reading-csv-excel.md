# 📂 02 — Reading CSV and Excel Files
## Loading Real Data into Pandas

> [!info] Goal
> Learn how to read common tabular file formats, inspect loaded data, and save results back to disk.

---

## Why File Reading Matters

Most Data Science work starts with data stored in files:

- CSV exports from apps
- Excel sheets from business teams
- downloaded datasets
- logs and reports
- data shared by stakeholders

Pandas makes these files easy to load into DataFrames.

```python
import pandas as pd
```

---

## Reading CSV Files

```python
df = pd.read_csv("students.csv")
```

This loads `students.csv` into a DataFrame.

### Example CSV

```csv
name,department,salary,experience
Alice,Sales,50000,2
Bob,Tech,80000,5
Charlie,Tech,75000,4
Diana,HR,45000,1
```

### Load and Inspect

```python
employees = pd.read_csv("employees.csv")

print(employees.head())
print(employees.shape)
print(employees.info())
```

---

## Useful `read_csv()` Parameters

| Parameter | Purpose |
|-----------|---------|
| `sep` | Separator, e.g. comma, tab, semicolon |
| `header` | Which row contains column names |
| `names` | Manually provide column names |
| `usecols` | Load only selected columns |
| `nrows` | Load only first N rows |
| `skiprows` | Skip rows at the top |
| `na_values` | Treat custom values as missing |
| `parse_dates` | Convert columns to datetime |

### Examples

```python
pd.read_csv("data.csv", sep=",")
pd.read_csv("data.tsv", sep="\t")
pd.read_csv("data.csv", usecols=["name", "salary"])
pd.read_csv("data.csv", nrows=100)
pd.read_csv("data.csv", na_values=["NA", "missing", "-"])
```

---

## CSV Without Headers

```csv
Alice,Sales,50000
Bob,Tech,80000
Charlie,Tech,75000
```

Read it with custom column names:

```python
df = pd.read_csv(
    "employees_no_header.csv",
    header=None,
    names=["name", "department", "salary"]
)
```

---

## Reading Excel Files

```python
df = pd.read_excel("employees.xlsx")
```

If the file has multiple sheets:

```python
sales = pd.read_excel("company.xlsx", sheet_name="Sales")
hr = pd.read_excel("company.xlsx", sheet_name="HR")
```

Read all sheets:

```python
sheets = pd.read_excel("company.xlsx", sheet_name=None)

print(sheets.keys())
```

`sheets` is a dictionary where sheet names are keys and DataFrames are values.

---

## Excel Engine Note

For `.xlsx` files, Pandas commonly uses `openpyxl`.

Install it if needed:

```bash
pip install openpyxl
```

---

## File Paths

### Same Folder

```python
df = pd.read_csv("employees.csv")
```

### Inside a Data Folder

```python
df = pd.read_csv("data/employees.csv")
```

### Absolute Path

```python
df = pd.read_csv(r"C:\data\employees.csv")
```

> [!tip] Windows Paths
> Use a raw string with `r"..."` or forward slashes to avoid escape character problems.

---

## Inspect Immediately After Loading

Always run these after loading a file:

```python
print(df.head())
print(df.shape)
print(df.columns)
print(df.dtypes)
print(df.isna().sum())
```

This helps you catch:

- wrong separators
- missing headers
- unexpected column names
- incorrect data types
- missing values

---

## Saving Data

### Save to CSV

```python
df.to_csv("clean_employees.csv", index=False)
```

`index=False` prevents Pandas from writing the row index as an extra column.

### Save to Excel

```python
df.to_excel("clean_employees.xlsx", index=False)
```

### Save Selected Columns

```python
df[["name", "salary"]].to_csv("employee_salaries.csv", index=False)
```

---

## Mini Workflow

```python
import pandas as pd

employees = pd.read_csv("employees.csv")

print(employees.head())
print(employees.info())

high_salary = employees[employees["salary"] >= 70000]

high_salary.to_csv("high_salary_employees.csv", index=False)
```

---

## Common Errors

### File Not Found

```text
FileNotFoundError: [Errno 2] No such file or directory
```

Check:

- spelling
- folder location
- current working directory
- file extension

### Wrong Separator

If the data loads into one giant column, the separator is probably wrong.

```python
df = pd.read_csv("data.tsv", sep="\t")
df = pd.read_csv("data_semicolon.csv", sep=";")
```

### Extra Index Column

If you see `Unnamed: 0`, the file was probably saved with an index.

```python
df = pd.read_csv("file.csv", index_col=0)
```

Better when saving:

```python
df.to_csv("file.csv", index=False)
```

---

## Practice

Create a file named `sales.csv`:

```csv
product,category,quantity,price
Laptop,Electronics,2,75000
Mouse,Accessories,5,600
Keyboard,Accessories,3,1200
Monitor,Electronics,2,15000
```

Tasks:

- read the file using `pd.read_csv()`
- print first 3 rows
- print shape and data types
- create a new column `revenue = quantity * price`
- save the result as `sales_with_revenue.csv`

---

## ✅ Key Takeaways

- Use `pd.read_csv()` for CSV files.
- Use `pd.read_excel()` for Excel files.
- Inspect data immediately after loading.
- Use `usecols`, `nrows`, `sep`, and `na_values` when needed.
- Use `to_csv(..., index=False)` when saving clean files.

---

## 🔗 What's Next?

➡️ [[03-filtering-and-sorting]] — Select the rows you actually need
