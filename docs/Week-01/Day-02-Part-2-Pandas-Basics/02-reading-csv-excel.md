# Reading CSV and Excel Files

Every real analysis starts with getting data into a DataFrame. You will rarely build DataFrames from scratch in production — you will load them from files, databases, or APIs. Getting this step right means your downstream analysis does not start on a broken foundation.

## Learning Objectives

- Use `pd.read_csv()` with full control over separators, headers, column selection, types, and missing value handling
- Handle common messy CSV problems: wrong separator, no header, mixed encodings, extra index column
- Load Excel files with sheet selection using `pd.read_excel()`
- Inspect a DataFrame immediately after loading to catch problems early
- Save DataFrames to CSV and Excel with correct index handling
- Load large files efficiently using `nrows` and `chunksize`

---

## Why This Step Matters More Than You Think

A wrong `sep` parameter loads your entire file as one column. An unspecified `encoding` raises an error on international characters. An unparsed date column stays as a string and breaks every time-series calculation you attempt. Most analysis bugs are not in the analysis — they are in the loading step, discovered two hours later.

The habit: read the file, immediately inspect, verify before proceeding.

---

## Reading CSV Files

```python
import pandas as pd

employees = pd.read_csv("employees.csv")

print(employees.head())
print(employees.shape)
print(employees.info())
```

### What a clean CSV looks like

```csv
name,department,salary,years_exp,city
Priya,Sales,52000,2,Delhi
Rohan,Engineering,88000,6,Bangalore
Amit,Engineering,79000,4,Pune
Divya,HR,46000,1,Delhi
Karan,Sales,67000,3,Mumbai
```

Pandas infers the separator (`,`), uses the first row as column names, and assigns dtypes automatically.

---

## The Key Parameters You Actually Need

Most tutorials show `pd.read_csv("file.csv")` and stop. In real work you need these:

### sep — the separator

```python
# Tab-separated files (.tsv or sometimes .txt)
df = pd.read_csv("data.tsv", sep="\t")

# Semicolons (common in European locale CSV exports)
df = pd.read_csv("data_eu.csv", sep=";")

# Pipe-delimited (common in data warehouse exports)
df = pd.read_csv("data.csv", sep="|")
```

> [!warning] One giant column means wrong separator
> If your DataFrame has one column with a name like `"name,department,salary,years_exp"`, the separator is wrong. Check the actual file and pass the correct `sep` value.

### header and names — controlling column names

```python
# File with no header row — first row is data, not column names
df = pd.read_csv(
    "employees_raw.csv",
    header=None,
    names=["name", "department", "salary", "years_exp", "city"]
)

# Skip the first 2 rows (metadata/comments before headers)
df = pd.read_csv("report.csv", skiprows=2)

# Header is on row 3, not row 1 (0-indexed)
df = pd.read_csv("report.csv", header=2)
```

### index_col — setting the row index from a column

```python
# Use 'name' as the row index instead of 0, 1, 2...
df = pd.read_csv("employees.csv", index_col="name")

# Or use the first column as index (by position)
df = pd.read_csv("employees.csv", index_col=0)
```

> [!tip] When to use index_col
> If your file was previously saved with `df.to_csv("file.csv")` (without `index=False`), it will have an `Unnamed: 0` column on reload. Fix it with `index_col=0` on read, or prevent it with `index=False` on write.

### usecols — load only the columns you need

```python
# Load only two of the five columns
df = pd.read_csv(
    "employees.csv",
    usecols=["name", "salary"]
)

# Or specify by position (columns 0 and 2)
df = pd.read_csv("employees.csv", usecols=[0, 2])
```

Loading fewer columns reduces memory usage significantly on wide datasets.

### dtype — force column types at load time

```python
# Prevent pandas from auto-inferring a product_id as integer
df = pd.read_csv(
    "orders.csv",
    dtype={
        "product_id": str,          # keep as string even if it looks numeric
        "customer_id": str,
        "amount": float
    }
)
```

> [!warning] Auto-inferred dtypes are not always correct
> Pandas reads the first N rows to guess dtypes. A column full of integers in the first 1000 rows but with a string "N/A" in row 1001 will fail mid-load or silently coerce values. Specify `dtype` explicitly for columns you know the type of.

### na_values — teach pandas what "missing" looks like

```python
# These strings should all be treated as NaN
df = pd.read_csv(
    "survey.csv",
    na_values=["N/A", "n/a", "NA", "none", "None", "-", "?", ""]
)
```

By default, pandas treats blank cells, `NaN`, `NA`, and a few others as missing. Your data may use something different.

### parse_dates — convert date columns on load

```python
# Tell pandas which columns contain dates
df = pd.read_csv(
    "orders.csv",
    parse_dates=["order_date", "ship_date"]
)

print(df["order_date"].dtype)   # datetime64[ns]
```

Without `parse_dates`, date columns load as `object` (string). Every date operation you attempt will fail or return wrong results.

### nrows — load a sample of a large file

```python
# Load only the first 500 rows to explore before committing to full load
df_sample = pd.read_csv("large_dataset.csv", nrows=500)

print(df_sample.shape)   # (500, n_cols)
```

### chunksize — process large files in pieces

```python
# Process a 5 million row file in chunks of 100,000
chunk_results = []

for chunk in pd.read_csv("large_sales.csv", chunksize=100_000):
    # Do work on each chunk — filter, aggregate, etc.
    high_value = chunk[chunk["amount"] > 10000]
    chunk_results.append(high_value)

result = pd.concat(chunk_results, ignore_index=True)
```

> [!info] When to use chunksize
> If a file is too large to fit in memory, `chunksize` lets you process it piece by piece. For files that fit in memory, load them whole — iterating chunks is slower when memory is not a constraint.

---

## File Encoding

Encoding issues are common when files come from different operating systems or international sources.

```python
# Try utf-8 first (works for most modern files)
df = pd.read_csv("data.csv", encoding="utf-8")

# Latin-1 covers Western European characters (common in older Windows exports)
df = pd.read_csv("data.csv", encoding="latin-1")

# Windows-1252 (another common Windows encoding)
df = pd.read_csv("data.csv", encoding="cp1252")
```

> [!warning] UnicodeDecodeError means wrong encoding
> If you see `UnicodeDecodeError: 'utf-8' codec can't decode byte...`, try `encoding="latin-1"`. It almost always works as a fallback, though it may not render special characters correctly. The right fix is to identify the file's actual encoding and specify it explicitly.

---

## Reading Excel Files

```python
# Read the first (default) sheet
df = pd.read_excel("company_data.xlsx")

# Read a specific sheet by name
sales_df = pd.read_excel("company_data.xlsx", sheet_name="Q1 Sales")

# Read a specific sheet by position (0 = first)
hr_df = pd.read_excel("company_data.xlsx", sheet_name=1)

# Read all sheets — returns a dict of {sheet_name: DataFrame}
all_sheets = pd.read_excel("company_data.xlsx", sheet_name=None)

print(list(all_sheets.keys()))     # ['Q1 Sales', 'Q2 Sales', 'HR', 'Finance']
q1 = all_sheets["Q1 Sales"]
```

`read_excel()` supports most of the same parameters as `read_csv()`: `header`, `index_col`, `usecols`, `dtype`, `na_values`, `nrows`.

> [!info] Required engine for .xlsx files
> Pandas uses `openpyxl` to read `.xlsx` files and `xlrd` for older `.xls` files.
>
> ```bash
> pip install openpyxl   # for .xlsx
> pip install xlrd       # for .xls  (old format)
> ```

---

## Inspect Immediately After Loading

Make this a reflex. Every time you load a file, run this block before doing anything else:

```python
orders = pd.read_csv("orders.csv")

print(orders.shape)           # how many rows and columns
print(orders.head())          # does the data look right?
print(orders.dtypes)          # are types what you expect?
print(orders.isna().sum())    # how many missing values per column?
print(orders.columns.tolist()) # column names (catch typos and extra spaces)
```

> [!tip] Column names with leading/trailing spaces are invisible bugs
> If a column loads as `" salary"` (with a space), `df["salary"]` raises a `KeyError`. Catch it early:
>
> ```python
> df.columns = df.columns.str.strip()
> ```
>
> This removes leading and trailing whitespace from all column names in one line.

---

## Saving Data

### Save to CSV

```python
# Always use index=False unless your index carries meaning
cleaned_orders = orders.dropna()
cleaned_orders.to_csv("orders_clean.csv", index=False)
```

### Save selected columns only

```python
orders[["order_id", "customer_name", "amount"]].to_csv(
    "order_summary.csv", index=False
)
```

### Save to Excel

```python
cleaned_orders.to_excel("orders_clean.xlsx", index=False)

# Save multiple DataFrames to different sheets in the same file
with pd.ExcelWriter("report.xlsx") as writer:
    orders.to_excel(writer, sheet_name="Orders", index=False)
    orders.groupby("category")["amount"].sum().to_excel(
        writer, sheet_name="By Category"
    )
```

---

## Common Problems and Fixes

| Symptom | Likely cause | Fix |
|---|---|---|
| One column with all data jammed together | Wrong separator | Add `sep="\t"` or `sep=";"` |
| `Unnamed: 0` column | File saved with index | Use `index_col=0` on read or `index=False` on write |
| Date column is `object` dtype | Dates not parsed | Add `parse_dates=["date_col"]` |
| `KeyError` on column you can see | Column name has spaces | `df.columns = df.columns.str.strip()` |
| `UnicodeDecodeError` | Wrong encoding | Try `encoding="latin-1"` |
| Numeric column loaded as `object` | Mixed values (e.g. "N/A" as string) | Add to `na_values`, then check dtype |
| `FileNotFoundError` | Wrong path | Print `import os; print(os.getcwd())` to check current directory |

---

## A Complete Loading Workflow

```python
import pandas as pd

# Load with full control
sales = pd.read_csv(
    "raw_sales_2024.csv",
    sep=",",
    usecols=["order_id", "product", "category", "quantity", "unit_price", "order_date", "city"],
    dtype={
        "order_id": str,
        "product": str,
        "category": str,
        "city": str,
    },
    parse_dates=["order_date"],
    na_values=["N/A", "?", "-", ""],
    encoding="utf-8"
)

# Immediately inspect
print(sales.shape)
print(sales.dtypes)
print(sales.isna().sum())

# Fix column name whitespace if present
sales.columns = sales.columns.str.strip()

# Add derived column
sales["revenue"] = sales["quantity"] * sales["unit_price"]

# Save clean version
sales.to_csv("sales_clean.csv", index=False)
```

---

> [!success] Key takeaways
> - `pd.read_csv()` has parameters for almost every messy file problem you will encounter — learn them, do not work around them.
> - Always inspect shape, dtypes, and missing values immediately after loading.
> - `parse_dates` on load saves you from hours of debugging date operations.
> - Save with `index=False` unless the index carries meaning.
> - When a file is too large for memory, use `chunksize` to process it incrementally.

---

[[01-series-and-dataframes|Previous: Series and DataFrames]] | [[03-filtering-and-sorting|Next: Filtering and Sorting]]
