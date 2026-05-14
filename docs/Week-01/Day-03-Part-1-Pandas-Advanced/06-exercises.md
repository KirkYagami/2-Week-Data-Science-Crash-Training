# 🧪 06 — Exercises: Pandas Advanced

> [!info] Goal
> Practice `groupby()`, merges, custom functions, missing values, and real-world cleaning workflows.

---

## Setup

```python
import pandas as pd
import numpy as np
```

Try each exercise before checking the solution.

---

## Exercise 1 — GroupBy Sales Summary

Create the dataset:

```python
sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Accessories", "Electronics"],
    "product": ["Laptop", "Mouse", "Laptop", "Keyboard", "Mouse", "Monitor"],
    "quantity": [2, 5, 1, 3, 10, 2],
    "price": [75000, 600, 75000, 1200, 600, 15000]
})

sales["revenue"] = sales["quantity"] * sales["price"]
```

Tasks:

- total revenue by city
- total revenue by category
- quantity sold by product
- category summary with total revenue, average revenue, and order count

### Solution

```python
print(sales.groupby("city")["revenue"].sum())
print(sales.groupby("category")["revenue"].sum())
print(sales.groupby("product")["quantity"].sum())

summary = sales.groupby("category").agg(
    total_revenue=("revenue", "sum"),
    average_revenue=("revenue", "mean"),
    order_count=("order_id", "count")
)

print(summary)
```

---

## Exercise 2 — Merge Orders with Customers and Products

Create:

```python
orders = pd.DataFrame({
    "order_id": [101, 102, 103, 104],
    "customer_id": [1, 2, 1, 3],
    "product_id": [10, 11, 12, 10],
    "quantity": [2, 1, 3, 1]
})

customers = pd.DataFrame({
    "customer_id": [1, 2, 3],
    "customer_name": ["Alice", "Bob", "Charlie"],
    "city": ["Delhi", "Mumbai", "Pune"]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200]
})
```

Tasks:

- merge orders with customers
- merge result with products
- calculate revenue
- calculate revenue by city

### Solution

```python
full_orders = (
    orders
    .merge(customers, on="customer_id", how="left", validate="many_to_one")
    .merge(products, on="product_id", how="left", validate="many_to_one")
)

full_orders["revenue"] = full_orders["quantity"] * full_orders["price"]

print(full_orders)
print(full_orders.groupby("city")["revenue"].sum())
```

---

## Exercise 3 — Apply and Vectorize

Create:

```python
employees = pd.DataFrame({
    "name": [" alice ", "BOB", "Charlie", "diana"],
    "department": ["Sales", "Tech", "Tech", "HR"],
    "salary": [50000, 80000, 75000, 45000],
    "experience": [2, 5, 4, 1],
    "rating": [4.2, 4.8, 4.5, 3.9]
})
```

Tasks:

- clean names to title case
- map departments to codes: Sales = S, Tech = T, HR = H
- create `salary_level`: High if salary >= 75000, Medium if >= 50000, else Low
- create `performance` using rating and experience

### Solution

```python
employees["name"] = employees["name"].str.strip().str.title()

employees["department_code"] = employees["department"].map({
    "Sales": "S",
    "Tech": "T",
    "HR": "H"
})

employees["salary_level"] = np.select(
    [
        employees["salary"] >= 75000,
        employees["salary"] >= 50000
    ],
    ["High", "Medium"],
    default="Low"
)

employees["performance"] = np.select(
    [
        (employees["rating"] >= 4.5) & (employees["experience"] >= 4),
        employees["rating"] >= 4.0
    ],
    ["Top Performer", "Good"],
    default="Needs Support"
)

print(employees)
```

---

## Exercise 4 — Missing Values

Create:

```python
customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5],
    "name": ["Alice", "Bob", None, "Diana", "Evan"],
    "age": [25, np.nan, 32, 29, np.nan],
    "city": ["Delhi", "Mumbai", None, "Pune", "Delhi"],
    "spend": [1200, 2500, np.nan, 1800, 0]
})
```

Tasks:

- count missing values
- create `age_was_missing`
- fill missing age with median
- fill missing city with `"Unknown"`
- drop rows missing name

### Solution

```python
print(customers.isna().sum())

customers["age_was_missing"] = customers["age"].isna()
customers["age"] = customers["age"].fillna(customers["age"].median())
customers["city"] = customers["city"].fillna("Unknown")
customers = customers.dropna(subset=["name"])

print(customers)
```

---

## Exercise 5 — Real-World Cleaning Pipeline

Create:

```python
raw = pd.DataFrame({
    " Customer ID ": [1, 2, 3, 3, 4],
    "Name": [" alice ", "BOB", "Charlie", "Charlie", None],
    "Age": ["25", "unknown", "32", "32", "-5"],
    "City": ["delhi", "Mumbai ", "", "Mumbai", "PUNE"],
    "Spend": ["1,200", "2500", "", "2500", "-100"]
})
```

Tasks:

- standardize column names
- remove duplicate customer IDs
- clean names and cities
- convert placeholders to missing values
- convert age and spend to numeric
- replace negative age and spend with missing
- fill missing values using sensible defaults

### Solution

```python
cleaned = raw.copy()

cleaned.columns = (
    cleaned.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)

cleaned = cleaned.drop_duplicates(subset=["customer_id"], keep="first")
cleaned = cleaned.replace({"": np.nan, "unknown": np.nan})

cleaned["name"] = cleaned["name"].str.strip().str.title()
cleaned["city"] = cleaned["city"].str.strip().str.title()

cleaned["age"] = pd.to_numeric(cleaned["age"], errors="coerce")
cleaned["spend"] = (
    cleaned["spend"]
    .str.replace(",", "", regex=False)
    .pipe(pd.to_numeric, errors="coerce")
)

cleaned.loc[cleaned["age"] < 0, "age"] = np.nan
cleaned.loc[cleaned["spend"] < 0, "spend"] = np.nan

cleaned["name"] = cleaned["name"].fillna("Unknown")
cleaned["city"] = cleaned["city"].fillna("Unknown")
cleaned["age"] = cleaned["age"].fillna(cleaned["age"].median())
cleaned["spend"] = cleaned["spend"].fillna(cleaned["spend"].median())

print(cleaned)
```

---

## Final Challenge — Customer Revenue Report

Use all skills together.

```python
orders = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105],
    "customer_id": [1, 2, 1, 3, 4],
    "product_id": [10, 11, 12, 10, 99],
    "quantity": [2, 1, 3, 1, 2]
})

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "customer_name": [" alice ", "BOB", None, "Diana"],
    "city": ["delhi", "Mumbai", "Pune", ""]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200]
})
```

Tasks:

- clean customer names and city
- merge orders with customers and products
- identify orders with missing product information
- calculate revenue
- create revenue by city
- create revenue by customer
- save final merged report

### One Possible Solution

```python
customers_clean = customers.copy()
customers_clean = customers_clean.replace({"": np.nan})
customers_clean["customer_name"] = (
    customers_clean["customer_name"]
    .str.strip()
    .str.title()
    .fillna("Unknown")
)
customers_clean["city"] = (
    customers_clean["city"]
    .str.strip()
    .str.title()
    .fillna("Unknown")
)

report = (
    orders
    .merge(customers_clean, on="customer_id", how="left", validate="many_to_one")
    .merge(products, on="product_id", how="left", validate="many_to_one")
)

missing_products = report[report["product_name"].isna()]
print("Missing product rows:")
print(missing_products)

report["price"] = report["price"].fillna(0)
report["revenue"] = report["quantity"] * report["price"]

print("Revenue by city:")
print(report.groupby("city")["revenue"].sum())

print("Revenue by customer:")
print(report.groupby("customer_name")["revenue"].sum().sort_values(ascending=False))

report.to_csv("customer_revenue_report.csv", index=False)
```

---

## ✅ Self-Check

You are ready to move on if you can:

- [ ] summarize data with `groupby()`
- [ ] use named aggregations
- [ ] merge multiple DataFrames safely
- [ ] explain join types
- [ ] use vectorized logic before `apply()`
- [ ] handle missing values thoughtfully
- [ ] clean messy column names and text values
- [ ] convert numeric and date columns safely
- [ ] validate data after cleaning

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[05-real-world-data-cleaning]] | [[../Day-03-Part-2-Data-Visualization/01-matplotlib-basics|Data Visualization]] |

---

*Tags: #pandas #groupby #merge #apply #missing-values #data-cleaning*
