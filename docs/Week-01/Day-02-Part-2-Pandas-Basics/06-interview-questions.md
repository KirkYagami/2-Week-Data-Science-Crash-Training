# Interview Questions: Pandas Basics

These are the questions that come up in every data science interview at the junior-to-mid level. The collapsible answers are model answers — study them, but rephrase in your own words. Interviewers can tell when someone is reciting a script.

---

## Q1 — What is the difference between a Series and a DataFrame?

??? "Show answer"
    A `Series` is a one-dimensional labeled array — it has one index and one column of values. A `DataFrame` is a two-dimensional table with a shared index across all columns, where each column is a `Series`. Every column you extract from a DataFrame is a `Series`.

    ```python
    import pandas as pd

    # Series — 1D, one index
    scores = pd.Series([88, 92, 75], index=["Priya", "Rohan", "Amit"])

    # DataFrame — 2D, shared index
    df = pd.DataFrame({
        "score": [88, 92, 75],
        "grade": ["B+", "A", "B"]
    }, index=["Priya", "Rohan", "Amit"])

    # Each column is a Series
    print(type(df["score"]))   # <class 'pandas.core.series.Series'>
    ```

    The key distinction to mention in an interview: they both have a labeled index; the DataFrame adds named columns on top.

---

## Q2 — What is the pandas Index and why does it matter?

??? "Show answer"
    The Index is the row labels of a DataFrame or Series. It is not just a row counter — it is a first-class object that enables alignment, fast lookup, and time-series operations.

    ```python
    df = pd.DataFrame({
        "salary": [52000, 88000, 79000]
    }, index=["Priya", "Rohan", "Amit"])

    # Fast label-based lookup
    print(df.loc["Rohan"])   # salary    88000

    # When you add two DataFrames, pandas aligns by index
    a = pd.Series([1, 2, 3], index=["x", "y", "z"])
    b = pd.Series([10, 20, 30], index=["x", "z", "w"])
    print(a + b)
    # x    11.0  (matched)
    # y     NaN  (no match)
    # z    23.0  (matched)
    # w     NaN  (no match)
    ```

    This alignment behavior is why understanding the Index prevents bugs. Two DataFrames with mismatched indexes produce silent `NaN` values when combined.

---

## Q3 — What is the difference between `.loc` and `.iloc`?

??? "Show answer"
    `.loc` selects by **label** (row index labels and column names). `.iloc` selects by **integer position** (0-based, like Python lists).

    ```python
    import pandas as pd

    employees = pd.DataFrame({
        "name": ["Priya", "Rohan", "Amit"],
        "salary": [52000, 88000, 79000]
    })

    # .loc — by label
    print(employees.loc[0, "salary"])    # 52000 — row label 0, column "salary"
    print(employees.loc[0:1])            # rows with labels 0 AND 1 (both ends inclusive)

    # .iloc — by position
    print(employees.iloc[0, 1])          # first row, second column
    print(employees.iloc[0:1])           # only first row (end excluded, like Python slicing)
    ```

    The critical difference in slicing: `df.loc[0:3]` returns 4 rows (labels 0, 1, 2, 3). `df.iloc[0:3]` returns 3 rows (positions 0, 1, 2).

    The other critical case: after filtering, label 0 may no longer be at position 0. `.loc[0]` looks for the label `0`; `.iloc[0]` looks for whatever is first in the remaining rows. Use `reset_index(drop=True)` if you want positions and labels to align again.

---

## Q4 — What is `SettingWithCopyWarning` and how do you fix it?

??? "Show answer"
    `SettingWithCopyWarning` means pandas is unsure whether you are modifying the original DataFrame or a temporary copy. It appears when you filter a DataFrame and then try to modify the filtered result.

    ```python
    # This triggers the warning
    high_earners = employees[employees["salary"] > 60000]
    high_earners["bonus"] = 5000   # Warning — modification may be silently lost
    ```

    The root cause: when you filter a DataFrame, pandas may return a **view** (a window into the original) or a **copy** (an independent object). If it returns a view, your modification affects the original. If it returns a copy, it is silently discarded. Pandas is warning you that it cannot guarantee which behavior you get.

    **Fix 1: Use `.copy()` when you intend to work with a subset independently.**

    ```python
    high_earners = employees[employees["salary"] > 60000].copy()
    high_earners["bonus"] = 5000   # Safe — you own this copy
    ```

    **Fix 2: Use `.loc` to modify the original DataFrame directly.**

    ```python
    employees.loc[employees["salary"] > 60000, "bonus"] = 5000   # Safe
    ```

    The chained assignment pattern `df[condition]["col"] = value` is always wrong. Never use it.

---

## Q5 — What is the difference between a view and a copy in pandas?

??? "Show answer"
    A **view** is a reference to the original data — modifying the view modifies the original. A **copy** is an independent object — modifying it does not affect the original.

    Pandas does not always let you know which one you have. Some operations like `.iloc[:]` return views; others like boolean indexing may return either, depending on internal decisions.

    ```python
    # To guarantee you have a copy, call .copy() explicitly
    subset = df[df["salary"] > 60000].copy()

    # Now you can modify subset freely without affecting df
    subset["bonus"] = 5000
    ```

    In practice: whenever you filter a DataFrame and plan to modify the result, call `.copy()`. This makes your intent explicit and eliminates the warning.

---

## Q6 — Why can't you use Python's `and`, `or`, `not` with pandas conditions?

??? "Show answer"
    Python's `and`, `or`, and `not` operators expect a single boolean value (`True` or `False`). A pandas condition like `df["salary"] > 60000` returns a `Series` of boolean values — one per row. Python cannot determine the "truth value" of an entire Series, so it raises a `ValueError`.

    ```python
    # Raises ValueError: The truth value of a Series is ambiguous
    df[df["salary"] > 60000 and df["city"] == "Delhi"]

    # Correct: use bitwise operators with parentheses
    df[(df["salary"] > 60000) & (df["city"] == "Delhi")]   # AND
    df[(df["salary"] > 60000) | (df["city"] == "Delhi")]   # OR
    df[~(df["city"] == "Delhi")]                            # NOT
    ```

    The parentheses around each condition are mandatory because `&` and `|` have higher precedence than comparison operators. Without parentheses, Python evaluates `60000 & df["city"]` first, which is nonsensical.

---

## Q7 — What does `SettingWithCopyWarning` look like and what is `inplace=True`?

??? "Show answer"
    `SettingWithCopyWarning` is a runtime warning (not an error) that prints something like:

    ```
    SettingWithCopyWarning: A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    ```

    Regarding `inplace=True`: many pandas methods like `sort_values()`, `rename()`, `drop()`, and `fillna()` accept `inplace=True` to modify the DataFrame without returning a new one. Most experienced practitioners avoid it because:

    - It cannot be used in method chains
    - It triggers `SettingWithCopyWarning` on filtered DataFrames
    - It is confusing to readers who expect pandas to be immutable by default

    ```python
    # inplace=True style (avoid)
    df.sort_values("salary", inplace=True)

    # Preferred — explicit assignment
    df = df.sort_values("salary")
    ```

    Prefer assignment. It is unambiguous about what changes and works correctly in all contexts.

---

## Q8 — How do you handle missing values in pandas?

??? "Show answer"
    Missing values in pandas are represented as `NaN` (Not a Number) for numeric columns and `None` or `NaN` for object columns.

    **Detection:**

    ```python
    df.isna().sum()             # count per column
    df.isna().mean() * 100      # percentage per column
    df[df["salary"].isna()]     # rows where salary is missing
    ```

    **Handling strategies:**

    ```python
    # 1. Drop rows with any missing value
    df_clean = df.dropna()

    # 2. Drop rows where specific columns are missing
    df_clean = df.dropna(subset=["salary", "department"])

    # 3. Fill with a constant
    df["city"] = df["city"].fillna("Unknown")

    # 4. Fill with the mean (numeric columns)
    df["salary"] = df["salary"].fillna(df["salary"].mean())

    # 5. Fill with the median (better for skewed data)
    df["salary"] = df["salary"].fillna(df["salary"].median())

    # 6. Forward-fill (carry last valid value forward — common in time series)
    df["price"] = df["price"].ffill()
    ```

    The right strategy depends on the data and the question. Dropping rows is appropriate when missing data is random and few rows are affected. Filling with mean or median is appropriate when missing is random and you cannot afford to lose rows.

---

## Q9 — What key `pd.read_csv()` parameters do you use most often?

??? "Show answer"
    ```python
    pd.read_csv(
        "file.csv",
        sep=",",              # separator character (default is comma)
        header=0,             # row to use as column names (default is first row)
        index_col="id",       # column to use as row index
        usecols=["a", "b"],   # load only these columns
        dtype={"id": str},    # force column types (do not let pandas guess)
        na_values=["N/A", "?", "-"],  # strings to treat as NaN
        parse_dates=["date"], # parse these columns as datetime
        nrows=1000,           # load only first N rows (useful for large files)
        encoding="utf-8",     # file encoding (try latin-1 if utf-8 fails)
        skiprows=2,           # skip the first N rows
    )
    ```

    The most commonly needed ones:
    - `sep` — wrong separator is the most common reading error
    - `parse_dates` — dates as strings break all date math
    - `usecols` — loading only needed columns saves memory on large files
    - `na_values` — datasets use many different strings for "missing"
    - `encoding` — international data often fails with the default utf-8

---

## Q10 — What is the difference between `df["col"]` and `df[["col"]]`?

??? "Show answer"
    `df["col"]` (single brackets, string key) returns a **Series** — a one-dimensional object.

    `df[["col"]]` (double brackets, list key) returns a **DataFrame** — a two-dimensional object with one column.

    ```python
    import pandas as pd

    df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})

    col_series = df["a"]
    print(type(col_series))    # <class 'pandas.core.series.Series'>
    print(col_series.shape)    # (3,)

    col_df = df[["a"]]
    print(type(col_df))        # <class 'pandas.core.frame.DataFrame'>
    print(col_df.shape)        # (3, 1)
    ```

    This matters because many methods behave differently on a Series vs. a DataFrame. For example, `df.merge()` and `pd.concat()` work on DataFrames, not Series. If your code receives a Series when it expects a DataFrame, you will get shape errors or wrong results.

---

## Q11 — What does `.describe()` show and what are its limitations?

??? "Show answer"
    `.describe()` returns summary statistics for numeric columns: count, mean, standard deviation, min, 25th percentile (Q1), median (50th), 75th percentile (Q3), and max.

    ```python
    df.describe()
    df.describe(include="all")     # includes object and bool columns too
    df.describe(include="object")  # only non-numeric columns
    ```

    **Limitations to mention in an interview:**

    1. It only covers numeric columns by default. Categorical columns need `value_counts()`.
    2. The `count` row tells you how many non-null values exist, not total rows — so it is also a missing value indicator.
    3. Mean and standard deviation are sensitive to outliers. The median (50%) and IQR (Q3-Q1) are more robust measures for skewed data.
    4. It does not show distributions — two columns can have identical `describe()` output with completely different distributions (Anscombe's Quartet). Always plot alongside describe.

---

## Q12 — How do you sort a DataFrame by multiple columns with different sort directions?

??? "Show answer"
    Pass lists to `sort_values()` — one list for column names, one for the ascending flag per column.

    ```python
    import pandas as pd

    employees = pd.DataFrame({
        "name": ["Priya", "Rohan", "Amit", "Divya", "Karan"],
        "department": ["Sales", "Engineering", "Engineering", "HR", "Sales"],
        "salary": [52000, 88000, 79000, 46000, 67000]
    })

    # Sort department A-Z, then salary high-to-low within each department
    sorted_df = employees.sort_values(
        by=["department", "salary"],
        ascending=[True, False]
    )

    print(sorted_df)
    # Output:
    #     name   department  salary
    # 1  Rohan  Engineering   88000
    # 2   Amit  Engineering   79000
    # 3  Divya           HR   46000
    # 0  Priya        Sales   52000
    # 4  Karan        Sales   67000
    ```

    Note: `sort_values()` returns a new DataFrame. The original is not modified. Assign back if you need the sorted version to persist:

    ```python
    employees = employees.sort_values(["department", "salary"], ascending=[True, False])
    ```

---

## Quick Revision Checklist

- [ ] Series vs. DataFrame — dimensions, relationship, when you get each
- [ ] The Index — what it is, why it matters, how alignment works
- [ ] `.loc` vs `.iloc` — label vs. position, slice endpoint behavior
- [ ] `SettingWithCopyWarning` — what it means, `.copy()` fix, `.loc` fix
- [ ] View vs. copy — how to guarantee a copy
- [ ] Why `and`/`or` fail — use `&`, `|`, `~` with parentheses
- [ ] Missing value detection — `.isna().sum()`, `.info()`, `dropna=False`
- [ ] Missing value handling — `dropna()`, `fillna()`, `ffill()`, `bfill()`
- [ ] `pd.read_csv()` key parameters — `sep`, `parse_dates`, `usecols`, `na_values`, `encoding`
- [ ] `df["col"]` returns Series; `df[["col"]]` returns DataFrame
- [ ] `describe()` — what it shows and what its limitations are
- [ ] Multi-column sort with mixed ascending/descending

---

[[05-practice-exercises|Previous: Practice Exercises]] | [[../Day-03-Part-1-Pandas-Advanced/01-groupby|Next: Day 03 — Pandas Advanced]]
