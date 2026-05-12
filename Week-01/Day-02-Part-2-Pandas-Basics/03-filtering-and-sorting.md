# 🔎 03 — Filtering and Sorting
## Selecting the Rows That Matter

> [!info] Goal
> Learn how to select columns, filter rows using conditions, and sort data for analysis.

---

## Sample Dataset

Use this DataFrame for the examples:

```python
import pandas as pd

employees = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie", "Diana", "Evan"],
    "department": ["Sales", "Tech", "Tech", "HR", "Sales"],
    "salary": [50000, 80000, 75000, 45000, 62000],
    "experience": [2, 5, 4, 1, 3],
    "city": ["Delhi", "Bangalore", "Pune", "Delhi", "Mumbai"]
})
```

---

## Selecting Columns

```python
employees["name"]
employees[["name", "salary"]]
```

Use single brackets for one column and double brackets for multiple columns.

---

## Filtering Rows

### One Condition

```python
high_salary = employees[employees["salary"] > 60000]
```

### Multiple Conditions

```python
tech_high_salary = employees[
    (employees["department"] == "Tech") &
    (employees["salary"] > 70000)
]
```

Use:

- `&` for AND
- `|` for OR
- `~` for NOT

Always wrap each condition in parentheses.

### OR Condition

```python
sales_or_hr = employees[
    (employees["department"] == "Sales") |
    (employees["department"] == "HR")
]
```

### NOT Condition

```python
not_tech = employees[employees["department"] != "Tech"]
```

or:

```python
not_tech = employees[~(employees["department"] == "Tech")]
```

---

## Filtering with `.isin()`

Use `.isin()` when checking membership in a list.

```python
target_cities = ["Delhi", "Mumbai"]

filtered = employees[employees["city"].isin(target_cities)]
```

This is cleaner than writing multiple OR conditions.

---

## Filtering Text

```python
employees[employees["name"].str.startswith("A")]
employees[employees["name"].str.contains("a", case=False)]
employees[employees["city"].str.lower() == "delhi"]
```

> [!warning] Missing Values
> If text columns contain missing values, use `na=False` with string filters.

```python
employees[employees["name"].str.contains("a", case=False, na=False)]
```

---

## Filtering with `.loc`

`.loc` is excellent when filtering rows and selecting columns at the same time.

```python
employees.loc[
    employees["salary"] > 60000,
    ["name", "department", "salary"]
]
```

This means:

- rows where salary > 60000
- only selected columns

---

## Filtering with `.query()`

```python
employees.query("salary > 60000")
employees.query("department == 'Tech' and salary > 70000")
```

For column names with spaces, use backticks:

```python
df.query("`monthly salary` > 50000")
```

---

## Sorting Values

### Sort Ascending

```python
employees.sort_values("salary")
```

### Sort Descending

```python
employees.sort_values("salary", ascending=False)
```

### Sort by Multiple Columns

```python
employees.sort_values(
    ["department", "salary"],
    ascending=[True, False]
)
```

This sorts department A-Z, then salary high-to-low within each department.

---

## Top and Bottom Rows

```python
employees.nlargest(3, "salary")
employees.nsmallest(2, "salary")
```

These are often clearer than sorting and then using `.head()`.

---

## Resetting Index

After filtering or sorting, the old row labels remain.

```python
filtered = employees[employees["salary"] > 60000]
filtered = filtered.reset_index(drop=True)
```

`drop=True` prevents the old index from becoming a new column.

---

## Common Patterns

### Find High Performers

```python
high_experience_high_salary = employees[
    (employees["experience"] >= 3) &
    (employees["salary"] >= 70000)
]
```

### Select Columns After Filtering

```python
result = employees.loc[
    employees["department"] == "Tech",
    ["name", "salary"]
]
```

### Sort Filtered Data

```python
result = (
    employees[employees["salary"] > 50000]
    .sort_values("salary", ascending=False)
    .reset_index(drop=True)
)
```

---

## Common Mistakes

### Using `and` Instead of `&`

```python
# Wrong
employees[(employees["salary"] > 60000) and (employees["experience"] > 2)]

# Correct
employees[(employees["salary"] > 60000) & (employees["experience"] > 2)]
```

### Missing Parentheses

```python
# Wrong
employees[employees["salary"] > 60000 & employees["experience"] > 2]

# Correct
employees[(employees["salary"] > 60000) & (employees["experience"] > 2)]
```

### Expecting Sort to Modify Original Data

```python
employees.sort_values("salary")
```

The original `employees` is unchanged unless you assign:

```python
employees = employees.sort_values("salary")
```

---

## Practice

Using the `employees` DataFrame:

- filter employees from the `Tech` department
- filter employees with salary greater than 60000
- filter employees from Delhi or Mumbai
- select only `name` and `salary` for employees with experience >= 3
- sort employees by salary descending
- find the top 2 highest-paid employees
- reset the index after filtering

---

## ✅ Key Takeaways

- Filtering uses Boolean conditions.
- Use `&`, `|`, and `~` instead of `and`, `or`, and `not`.
- Use `.isin()` for checking multiple possible values.
- Use `.loc` to filter rows and select columns together.
- Use `sort_values()` for sorting.
- Use `nlargest()` and `nsmallest()` for quick top/bottom records.

---

## 🔗 What's Next?

➡️ [[04-basic-data-analysis]] — Turn rows and columns into insights
