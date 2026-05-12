# 🧪 05 — Practice Exercises: Pandas Basics

> [!info] Goal
> Practice creating DataFrames, reading files, filtering, sorting, and answering basic analysis questions.

---

## Setup

```python
import pandas as pd
```

Use a notebook or a `.py` file. Try each exercise before looking at the sample solution.

---

## Exercise 1 — Create a DataFrame

Create this employee DataFrame:

```python
employees = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie", "Diana", "Evan"],
    "department": ["Sales", "Tech", "Tech", "HR", "Sales"],
    "salary": [50000, 80000, 75000, 45000, 62000],
    "experience": [2, 5, 4, 1, 3],
    "city": ["Delhi", "Bangalore", "Pune", "Delhi", "Mumbai"]
})
```

Tasks:

- print the first 3 rows
- print shape
- print column names
- print data types
- select only `name`
- select `name`, `department`, and `salary`

### Solution

```python
print(employees.head(3))
print(employees.shape)
print(employees.columns)
print(employees.dtypes)
print(employees["name"])
print(employees[["name", "department", "salary"]])
```

---

## Exercise 2 — Add and Rename Columns

Using the `employees` DataFrame:

- create a column `bonus` equal to 10% of salary
- create a column `total_compensation` equal to salary + bonus
- rename `experience` to `years_experience`

### Solution

```python
employees["bonus"] = employees["salary"] * 0.10
employees["total_compensation"] = employees["salary"] + employees["bonus"]

employees = employees.rename(columns={
    "experience": "years_experience"
})

print(employees.head())
```

---

## Exercise 3 — Filtering Rows

Using the original employee data:

- filter employees with salary greater than 60000
- filter employees in the Tech department
- filter employees with experience at least 3 years
- filter employees from Delhi or Mumbai

### Solution

```python
high_salary = employees[employees["salary"] > 60000]
tech = employees[employees["department"] == "Tech"]
experienced = employees[employees["experience"] >= 3]
target_cities = employees[employees["city"].isin(["Delhi", "Mumbai"])]

print(high_salary)
print(tech)
print(experienced)
print(target_cities)
```

---

## Exercise 4 — Multiple Conditions

Find employees who:

- are in Tech and earn more than 70000
- are in Sales or HR
- are not from Delhi

### Solution

```python
tech_high_salary = employees[
    (employees["department"] == "Tech") &
    (employees["salary"] > 70000)
]

sales_or_hr = employees[
    (employees["department"] == "Sales") |
    (employees["department"] == "HR")
]

not_delhi = employees[employees["city"] != "Delhi"]

print(tech_high_salary)
print(sales_or_hr)
print(not_delhi)
```

---

## Exercise 5 — Sorting

Using the employee data:

- sort by salary ascending
- sort by salary descending
- sort by department ascending and salary descending
- find the top 2 highest-paid employees

### Solution

```python
print(employees.sort_values("salary"))
print(employees.sort_values("salary", ascending=False))

print(employees.sort_values(
    ["department", "salary"],
    ascending=[True, False]
))

print(employees.nlargest(2, "salary"))
```

---

## Exercise 6 — Read and Analyze CSV

Create a file named `sales.csv`:

```csv
order_id,product,category,quantity,price,city
101,Laptop,Electronics,2,75000,Delhi
102,Mouse,Accessories,5,600,Mumbai
103,Laptop,Electronics,1,75000,Delhi
104,Keyboard,Accessories,3,1200,Pune
105,Mouse,Accessories,10,600,Mumbai
106,Monitor,Electronics,2,15000,Bangalore
```

Tasks:

- read the CSV
- inspect first rows, shape, and data types
- create a `revenue` column
- find total revenue
- find average revenue
- save as `sales_analyzed.csv`

### Solution

```python
sales = pd.read_csv("sales.csv")

print(sales.head())
print(sales.shape)
print(sales.dtypes)

sales["revenue"] = sales["quantity"] * sales["price"]

print(sales["revenue"].sum())
print(sales["revenue"].mean())

sales.to_csv("sales_analyzed.csv", index=False)
```

---

## Exercise 7 — Basic Sales Questions

Using the `sales` DataFrame:

- count orders by product
- count orders by city
- find total quantity sold by product
- find total revenue by category
- find the highest revenue order

### Solution

```python
print(sales["product"].value_counts())
print(sales["city"].value_counts())
print(sales.groupby("product")["quantity"].sum())
print(sales.groupby("category")["revenue"].sum())
print(sales.nlargest(1, "revenue"))
```

---

## Exercise 8 — Missing Value Check

Create this DataFrame:

```python
customers = pd.DataFrame({
    "name": ["Amit", "Sara", "John", "Priya"],
    "age": [25, None, 32, 29],
    "city": ["Delhi", "Mumbai", None, "Pune"],
    "spend": [1200, 2500, None, 1800]
})
```

Tasks:

- print missing value counts
- print rows where age is missing
- calculate average spend ignoring missing values
- fill missing city with `"Unknown"`

### Solution

```python
print(customers.isna().sum())

missing_age = customers[customers["age"].isna()]
print(missing_age)

print(customers["spend"].mean())

customers["city"] = customers["city"].fillna("Unknown")
print(customers)
```

---

## Final Challenge — Mini Analysis Report

Using `sales.csv`, write code that prints:

- total number of orders
- total revenue
- average order revenue
- top product by order count
- top city by order count
- revenue by category
- top 3 orders by revenue

### One Possible Solution

```python
sales = pd.read_csv("sales.csv")
sales["revenue"] = sales["quantity"] * sales["price"]

print("Total orders:", len(sales))
print("Total revenue:", sales["revenue"].sum())
print("Average order revenue:", sales["revenue"].mean())
print("Top product:")
print(sales["product"].value_counts().head(1))
print("Top city:")
print(sales["city"].value_counts().head(1))
print("Revenue by category:")
print(sales.groupby("category")["revenue"].sum())
print("Top 3 orders:")
print(sales.nlargest(3, "revenue"))
```

---

## ✅ Self-Check

You are ready for Day 03 Pandas Advanced if you can:

- [ ] create a DataFrame from a dictionary
- [ ] read a CSV file
- [ ] select one or more columns
- [ ] filter rows with one condition
- [ ] filter rows with multiple conditions
- [ ] sort rows by a column
- [ ] create a new calculated column
- [ ] use `value_counts()`
- [ ] run simple `groupby()` summaries
- [ ] save a DataFrame to CSV

---

## 🔗 What's Next?

➡️ [[06-interview-questions]] — Review common Pandas basics interview questions
