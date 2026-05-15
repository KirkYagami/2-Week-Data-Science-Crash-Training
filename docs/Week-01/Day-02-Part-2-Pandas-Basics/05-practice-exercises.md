# Practice Exercises: Pandas Basics

The only way to get good at pandas is to write pandas code. Reading notes builds recognition; writing code builds fluency. These exercises are structured in three levels. Work through warm-up before main, and main before stretch. Try each exercise without looking at the solution first.

## Setup

```python
import pandas as pd
import numpy as np
```

---

## Warm-Up Exercises

These exercises check that you can use the core operations without hesitation.

### Warm-Up 1 — Build and Inspect a DataFrame

Create the following student grades DataFrame and answer the questions.

```python
grades = pd.DataFrame({
    "student_id": ["S001", "S002", "S003", "S004", "S005", "S006"],
    "name": ["Priya", "Rohan", "Amit", "Divya", "Karan", "Neha"],
    "section": ["A", "B", "A", "B", "A", "B"],
    "math_score": [88, 74, 92, 65, 79, 85],
    "english_score": [76, 81, 70, 88, 72, 90],
    "attendance_pct": [95.0, 82.0, 98.0, 74.0, 88.0, 91.0]
})
```

Tasks:

1. Print the first 3 rows.
2. Print the shape.
3. Print the dtypes.
4. Select only the `name` column.
5. Select `name`, `math_score`, and `english_score`.
6. Add a column `total_score` that is the sum of `math_score` and `english_score`.
7. Add a column `average_score` that is the average of the two scores.
8. Rename `attendance_pct` to `attendance`.

??? "Show answer"

    ```python
    # 1. First 3 rows
    print(grades.head(3))
    # Output:
    #   student_id   name section  math_score  english_score  attendance_pct
    # 0       S001  Priya       A          88             76            95.0
    # 1       S002  Rohan       B          74             81            82.0
    # 2       S003   Amit       A          92             70            98.0

    # 2. Shape
    print(grades.shape)
    # Output: (6, 6)

    # 3. Data types
    print(grades.dtypes)
    # Output:
    # student_id        object
    # name              object
    # section           object
    # math_score         int64
    # english_score      int64
    # attendance_pct   float64

    # 4. Single column (returns Series)
    print(grades["name"])

    # 5. Multiple columns (returns DataFrame)
    print(grades[["name", "math_score", "english_score"]])

    # 6. Total score
    grades["total_score"] = grades["math_score"] + grades["english_score"]

    # 7. Average score
    grades["average_score"] = grades["total_score"] / 2

    # 8. Rename column
    grades = grades.rename(columns={"attendance_pct": "attendance"})

    print(grades.head())
    ```

---

### Warm-Up 2 — .loc and .iloc Practice

Using the `grades` DataFrame from Warm-Up 1:

1. Use `.loc` to select the row with label `2` (Amit's row).
2. Use `.iloc` to select the third row (same row, using position).
3. Use `.loc` to select rows 0 through 3, columns `name` and `math_score`.
4. Use `.iloc` to select the first 3 rows and the first 2 columns.
5. Use `.loc` to select all rows where `section == "A"`, keeping only `name` and `average_score`.

??? "Show answer"

    ```python
    # 1. Row with label 2 using .loc
    print(grades.loc[2])
    # Output: student_id    S003, name    Amit, ...

    # 2. Third row using .iloc (position 2 = third row, 0-indexed)
    print(grades.iloc[2])

    # 3. Rows 0-3, specific columns with .loc (inclusive on both ends)
    print(grades.loc[0:3, ["name", "math_score"]])
    # Output:
    #     name  math_score
    # 0  Priya          88
    # 1  Rohan          74
    # 2   Amit          92
    # 3  Divya          65

    # 4. First 3 rows, first 2 columns with .iloc (end excluded)
    print(grades.iloc[0:3, 0:2])
    # Output:
    #   student_id   name
    # 0       S001  Priya
    # 1       S002  Rohan
    # 2       S003   Amit

    # 5. Filter + column selection with .loc
    section_a = grades.loc[grades["section"] == "A", ["name", "average_score"]]
    print(section_a)
    # Output:
    #     name  average_score
    # 0  Priya           82.0
    # 2   Amit           81.0
    # 4  Karan           75.5
    ```

---

### Warm-Up 3 — Basic Filtering

Using the `grades` DataFrame:

1. Filter students with `math_score` above 80.
2. Filter students in Section B.
3. Filter students with `attendance` below 85.
4. Filter students in Section A AND with `math_score` above 85.
5. Filter students in Section A OR with `english_score` above 85.
6. Find the top 2 students by `math_score`.

??? "Show answer"

    ```python
    # 1. Math score above 80
    print(grades[grades["math_score"] > 80])

    # 2. Section B
    print(grades[grades["section"] == "B"])

    # 3. Attendance below 85
    print(grades[grades["attendance"] < 85])

    # 4. Section A AND high math score
    high_math_a = grades[
        (grades["section"] == "A") &
        (grades["math_score"] > 85)
    ]
    print(high_math_a[["name", "section", "math_score"]])
    # Output:
    #     name section  math_score
    # 0  Priya       A          88
    # 2   Amit       A          92

    # 5. Section A OR high English score
    result = grades[
        (grades["section"] == "A") |
        (grades["english_score"] > 85)
    ]
    print(result[["name", "section", "english_score"]])

    # 6. Top 2 by math score
    print(grades.nlargest(2, "math_score")[["name", "math_score"]])
    # Output:
    #     name  math_score
    # 2   Amit          92
    # 0  Priya          88
    ```

---

## Main Exercises

These exercises require combining multiple operations and thinking through the right sequence.

### Main 1 — Sales Data Analysis

Work with this e-commerce dataset:

```python
orders = pd.DataFrame({
    "order_id": [f"ORD{1000+i}" for i in range(1, 16)],
    "customer_id": ["C001", "C002", "C003", "C001", "C004", "C002", "C005",
                    "C003", "C001", "C004", "C005", "C002", "C003", "C001", "C004"],
    "product": ["Laptop", "Headphones", "Mouse", "Keyboard", "Laptop", "Monitor",
                "Headphones", "Laptop", "Mouse", "Keyboard", "Monitor", "Mouse",
                "Headphones", "Keyboard", "Laptop"],
    "category": ["Electronics", "Electronics", "Accessories", "Accessories", "Electronics",
                 "Electronics", "Electronics", "Electronics", "Accessories", "Accessories",
                 "Electronics", "Accessories", "Electronics", "Accessories", "Electronics"],
    "units_sold": [1, 2, 5, 3, 1, 1, 2, 2, 8, 4, 1, 6, 1, 2, 1],
    "unit_price": [74999, 3499, 599, 1299, 74999, 16999, 3499, 74999, 599, 1299,
                   16999, 599, 3499, 1299, 74999],
    "city": ["Delhi", "Mumbai", "Bangalore", "Delhi", "Pune", "Mumbai", "Bangalore",
             "Delhi", "Mumbai", "Bangalore", "Pune", "Delhi", "Mumbai", "Pune", "Delhi"],
    "order_month": [1, 1, 1, 2, 2, 2, 3, 3, 3, 1, 1, 2, 2, 3, 3]
})
```

Tasks:

1. Add a `revenue` column (`units_sold * unit_price`).
2. Print the total revenue across all orders.
3. Print the average revenue per order.
4. Print the number of orders per category.
5. Find the 3 largest orders by revenue.
6. Filter orders where `revenue > 50000` and `city` is in `["Delhi", "Mumbai"]`.
7. Find the total revenue per city, sorted descending.
8. Find the total revenue per month.
9. Find how many unique customers placed orders.
10. Find which customer placed the most orders.

??? "Show answer"

    ```python
    # 1. Revenue column
    orders["revenue"] = orders["units_sold"] * orders["unit_price"]

    # 2. Total revenue
    print(f"Total revenue: ₹{orders['revenue'].sum():,}")
    # Output: Total revenue: ₹589,469

    # 3. Average revenue per order
    print(f"Average order revenue: ₹{orders['revenue'].mean():,.0f}")
    # Output: Average order revenue: ₹39,298

    # 4. Orders per category
    print(orders["category"].value_counts())
    # Output:
    # Electronics    9
    # Accessories    6

    # 5. Top 3 orders by revenue
    top_3 = orders.nlargest(3, "revenue")[["order_id", "product", "city", "revenue"]]
    print(top_3)

    # 6. High-value orders in target cities
    high_value_metro = orders[
        (orders["revenue"] > 50000) &
        (orders["city"].isin(["Delhi", "Mumbai"]))
    ]
    print(high_value_metro[["order_id", "customer_id", "product", "city", "revenue"]])

    # 7. Revenue by city (sorted descending)
    city_revenue = orders.groupby("city")["revenue"].sum().sort_values(ascending=False)
    print(city_revenue)

    # 8. Revenue by month
    monthly_revenue = orders.groupby("order_month")["revenue"].sum()
    print(monthly_revenue)

    # 9. Unique customer count
    print(f"Unique customers: {orders['customer_id'].nunique()}")
    # Output: Unique customers: 5

    # 10. Customer with most orders
    top_customer = orders["customer_id"].value_counts().head(1)
    print(top_customer)
    # Output: C001 = 4 orders
    ```

---

### Main 2 — Reading a CSV with Messy Properties

Create this CSV content (save it as `employees_messy.csv` or simulate with `io.StringIO`):

```python
import io

csv_content = """Employee ID,Full Name,Department,Monthly Salary,Join Date,Performance
E001,Priya Sharma,Sales,52000,2022-03-15,Good
E002,Rohan Mehta,Engineering,88000,2019-07-01,Excellent
E003,Amit Patel,Engineering,79000,2020-11-20,Good
E004,Divya Singh,HR,46000,2023-01-10,Average
E005,Karan Rao,Sales,67000,2021-05-25,Excellent
E006,Neha Gupta,Engineering,92000,2018-09-12,Excellent
E007,Suresh Kumar,HR,44000,2023-06-01,Good
E008,Anjali Desai,Sales,55000,2022-08-19,Average
"""

employees = pd.read_csv(
    io.StringIO(csv_content),
    parse_dates=["Join Date"]
)
```

Tasks:

1. Print `.info()` — note the column names.
2. Rename columns: `"Employee ID"` → `"emp_id"`, `"Full Name"` → `"name"`, `"Monthly Salary"` → `"monthly_salary"`, `"Join Date"` → `"join_date"`, `"Performance"` → `"performance"`, `"Department"` → `"department"`.
3. Sort employees by `monthly_salary` descending and print the top 3.
4. Filter employees in `"Engineering"` with `monthly_salary > 80000`.
5. Find the average salary per department.
6. Count how many employees are in each performance category.
7. Select only `name` and `monthly_salary` for employees who joined after 2021-01-01.
8. Add a column `annual_salary` equal to `monthly_salary * 12`.

??? "Show answer"

    ```python
    # 1. Info check
    employees.info()

    # 2. Rename columns
    employees = employees.rename(columns={
        "Employee ID": "emp_id",
        "Full Name": "name",
        "Department": "department",
        "Monthly Salary": "monthly_salary",
        "Join Date": "join_date",
        "Performance": "performance"
    })

    # 3. Top 3 by salary
    print(employees.nlargest(3, "monthly_salary")[["name", "department", "monthly_salary"]])
    # Output:
    #          name  department  monthly_salary
    # 5  Neha Gupta Engineering           92000
    # 1  Rohan Mehta Engineering           88000
    # 2   Amit Patel Engineering           79000

    # 4. Engineering with salary > 80000
    eng_high = employees[
        (employees["department"] == "Engineering") &
        (employees["monthly_salary"] > 80000)
    ]
    print(eng_high[["name", "monthly_salary"]])

    # 5. Average salary per department
    dept_avg = employees.groupby("department")["monthly_salary"].mean().round(0)
    print(dept_avg)

    # 6. Performance distribution
    print(employees["performance"].value_counts())

    # 7. Recent joiners — name and salary only
    recent_joiners = employees.loc[
        employees["join_date"] > "2021-01-01",
        ["name", "monthly_salary"]
    ]
    print(recent_joiners)

    # 8. Annual salary
    employees["annual_salary"] = employees["monthly_salary"] * 12
    print(employees[["name", "monthly_salary", "annual_salary"]].head())
    ```

---

### Main 3 — Missing Value Detection and Handling

```python
customer_survey = pd.DataFrame({
    "customer_id": ["C001", "C002", "C003", "C004", "C005", "C006", "C007", "C008"],
    "age": [28, None, 34, 45, None, 31, 27, 39],
    "city": ["Delhi", "Mumbai", None, "Pune", "Bangalore", None, "Delhi", "Mumbai"],
    "purchase_amount": [4500.0, 12000.0, 8900.0, None, 3200.0, 6700.0, None, 9100.0],
    "satisfaction_score": [4, 5, None, 3, 4, None, 5, 4],
    "membership_tier": ["Silver", "Gold", "Silver", None, "Bronze", "Gold", "Bronze", None]
})
```

Tasks:

1. Print missing value counts per column.
2. Print the percentage missing per column.
3. Print all rows that have at least one missing value.
4. Calculate the average `purchase_amount` (pandas skips NaN automatically — verify this).
5. Fill missing `city` values with `"Unknown"`.
6. Fill missing `age` with the median age.
7. Fill missing `satisfaction_score` with the mode (most common value).
8. Drop rows where `purchase_amount` is missing.
9. After all fills and drops, verify there are no more missing values.

??? "Show answer"

    ```python
    # 1. Missing counts
    print(customer_survey.isna().sum())
    # Output:
    # customer_id          0
    # age                  2
    # city                 2
    # purchase_amount      2
    # satisfaction_score   2
    # membership_tier      2

    # 2. Percentage missing
    missing_pct = customer_survey.isna().mean() * 100
    print(missing_pct.round(1))

    # 3. Rows with any missing value
    rows_with_na = customer_survey[customer_survey.isna().any(axis=1)]
    print(rows_with_na)

    # 4. Average purchase_amount (NaN skipped automatically)
    avg_purchase = customer_survey["purchase_amount"].mean()
    print(f"Average: ₹{avg_purchase:,.0f}")
    print(f"Count used in mean: {customer_survey['purchase_amount'].count()} of {len(customer_survey)}")

    # Work on a copy to avoid SettingWithCopyWarning
    survey = customer_survey.copy()

    # 5. Fill missing city
    survey["city"] = survey["city"].fillna("Unknown")

    # 6. Fill missing age with median
    age_median = survey["age"].median()
    survey["age"] = survey["age"].fillna(age_median)
    print(f"Age median used: {age_median}")

    # 7. Fill missing satisfaction_score with mode
    satisfaction_mode = survey["satisfaction_score"].mode()[0]
    survey["satisfaction_score"] = survey["satisfaction_score"].fillna(satisfaction_mode)

    # 8. Drop rows where purchase_amount is still missing
    survey = survey.dropna(subset=["purchase_amount"])

    # 9. Verify — should show all zeros
    print(survey.isna().sum())
    ```

---

## Stretch Exercises

These exercises require combining multiple techniques. There is no single correct solution — focus on getting correct output.

### Stretch 1 — Full Sales Analysis Pipeline

```python
store_sales = pd.DataFrame({
    "transaction_id": [f"TXN{1000+i}" for i in range(1, 21)],
    "date": pd.date_range("2024-01-01", periods=20, freq="3D"),
    "store_id": ["S01", "S02", "S01", "S03", "S02", "S01", "S03", "S02", "S01", "S03",
                 "S02", "S01", "S03", "S02", "S01", "S03", "S02", "S01", "S03", "S01"],
    "product_name": ["Rice 5kg", "Cooking Oil 1L", "Detergent 2kg", "Rice 5kg", "Sugar 1kg",
                     "Cooking Oil 1L", "Detergent 2kg", "Rice 5kg", "Sugar 1kg", "Cooking Oil 1L",
                     "Rice 5kg", "Detergent 2kg", "Sugar 1kg", "Cooking Oil 1L", "Rice 5kg",
                     "Detergent 2kg", "Sugar 1kg", "Cooking Oil 1L", "Rice 5kg", "Detergent 2kg"],
    "units": [10, 5, 8, 12, 20, 6, 15, 11, 25, 4, 9, 10, 18, 7, 14, 12, 22, 5, 16, 9],
    "unit_price": [250, 180, 320, 250, 85, 180, 320, 250, 85, 180,
                   250, 320, 85, 180, 250, 320, 85, 180, 250, 320],
    "store_city": ["Delhi", "Mumbai", "Delhi", "Bangalore", "Mumbai", "Delhi", "Bangalore",
                   "Mumbai", "Delhi", "Bangalore", "Mumbai", "Delhi", "Bangalore", "Mumbai",
                   "Delhi", "Bangalore", "Mumbai", "Delhi", "Bangalore", "Delhi"]
})
```

Tasks:

1. Add a `revenue` column.
2. Print a complete first-look inspection (shape, dtypes, missing values, describe).
3. Find total revenue per store (`store_id`).
4. Find which store has the highest average revenue per transaction.
5. Find the top 3 products by total revenue across all stores.
6. Filter transactions where `units > 15` and print store and product details.
7. Find which city generates the most total revenue.
8. Add a `high_volume` column: `True` if `units > 12`, `False` otherwise.
9. Sort the DataFrame by `date` ascending and by `revenue` descending within the same date.
10. Print a summary: total transactions, total revenue, unique products, unique stores.

??? "Show answer"

    ```python
    # 1. Revenue column
    store_sales["revenue"] = store_sales["units"] * store_sales["unit_price"]

    # 2. First-look inspection
    print(store_sales.shape)
    print(store_sales.dtypes)
    print(store_sales.isna().sum())
    print(store_sales.describe())

    # 3. Revenue per store
    store_revenue = store_sales.groupby("store_id")["revenue"].sum().sort_values(ascending=False)
    print(store_revenue)

    # 4. Highest average revenue per transaction by store
    store_avg = store_sales.groupby("store_id")["revenue"].mean().sort_values(ascending=False)
    print(f"Best store by average transaction value: {store_avg.index[0]}")

    # 5. Top 3 products by total revenue
    product_revenue = store_sales.groupby("product_name")["revenue"].sum().nlargest(3)
    print(product_revenue)

    # 6. High-unit transactions
    high_unit = store_sales[store_sales["units"] > 15][["store_id", "product_name", "units", "revenue"]]
    print(high_unit)

    # 7. Revenue by city
    city_revenue = store_sales.groupby("store_city")["revenue"].sum().sort_values(ascending=False)
    print(f"Top city: {city_revenue.index[0]} with ₹{city_revenue.iloc[0]:,}")

    # 8. High-volume flag
    store_sales["high_volume"] = np.where(store_sales["units"] > 12, True, False)

    # 9. Multi-column sort
    store_sales_sorted = store_sales.sort_values(
        ["date", "revenue"],
        ascending=[True, False]
    )
    print(store_sales_sorted[["date", "store_id", "product_name", "revenue"]].head(8))

    # 10. Summary
    print(f"Total transactions: {len(store_sales)}")
    print(f"Total revenue: ₹{store_sales['revenue'].sum():,}")
    print(f"Unique products: {store_sales['product_name'].nunique()}")
    print(f"Unique stores: {store_sales['store_id'].nunique()}")
    ```

---

### Stretch 2 — SettingWithCopyWarning Challenge

This exercise is about understanding views vs. copies — the most confusing pandas behavior for beginners.

```python
inventory = pd.DataFrame({
    "item_code": ["A01", "A02", "A03", "B01", "B02", "C01"],
    "item_name": ["Widget A", "Widget B", "Gadget C", "Tool D", "Tool E", "Part F"],
    "category": ["Widgets", "Widgets", "Gadgets", "Tools", "Tools", "Parts"],
    "stock_qty": [50, 0, 120, 30, 0, 80],
    "unit_cost": [250, 180, 490, 320, 140, 95]
})
```

Tasks:

1. Filter rows where `stock_qty == 0` and try to add a column `reorder_flag = True`. Observe whether you get a `SettingWithCopyWarning`.
2. Fix the issue using `.copy()`.
3. Fix the same operation using `.loc` directly on the original DataFrame instead.
4. Explain in a comment when you would choose `.copy()` vs `.loc`.

??? "Show answer"

    ```python
    # Task 1: This triggers SettingWithCopyWarning in most pandas versions
    out_of_stock = inventory[inventory["stock_qty"] == 0]
    out_of_stock["reorder_flag"] = True   # Warning here — may not affect original
    # The modification may be silently lost

    # Task 2: Fix with .copy() — you own the copy outright
    out_of_stock = inventory[inventory["stock_qty"] == 0].copy()
    out_of_stock["reorder_flag"] = True   # Safe — no warning
    print(out_of_stock)
    # Output shows reorder_flag = True for the zero-stock items

    # Task 3: Fix using .loc on the original
    inventory["reorder_flag"] = False   # Start with default value for all rows
    inventory.loc[inventory["stock_qty"] == 0, "reorder_flag"] = True
    print(inventory[["item_name", "stock_qty", "reorder_flag"]])
    # Output:
    #   item_name  stock_qty  reorder_flag
    # 0  Widget A         50         False
    # 1  Widget B          0          True
    # 2  Gadget C        120         False
    # 3    Tool D         30         False
    # 4    Tool E          0          True
    # 5    Part F         80         False

    # Task 4: When to use which approach:
    # Use .copy() when you need an independent subset to work with separately
    #   (e.g., you are building a report on just out-of-stock items)
    # Use .loc when you want to modify the original DataFrame in place
    #   (e.g., adding a flag column back into the main dataset)
    ```

---

## Self-Check

Before moving to Day 03 (Pandas Advanced), confirm you can do all of these without looking at notes:

- [ ] Create a DataFrame from a dictionary and from a list of dicts
- [ ] Use `.head()`, `.shape`, `.dtypes`, `.info()`, and `.isna().sum()` immediately after loading
- [ ] Select a single column (returning a Series) and multiple columns (returning a DataFrame)
- [ ] Use `.loc[row_condition, column_list]` for combined row filtering and column selection
- [ ] Use `.iloc[row_positions, col_positions]` for position-based access
- [ ] Filter with `&`, `|`, `~` and know why `and`/`or` do not work
- [ ] Use `.isin()` for membership filtering
- [ ] Use `.query()` for readable multi-condition filters
- [ ] Sort by one and multiple columns with mixed ascending/descending
- [ ] Use `.nlargest()` and `.nsmallest()` for top/bottom N
- [ ] Add calculated columns using vectorized expressions
- [ ] Use `.value_counts()`, `.nunique()`, `.describe()`, and `.corr()`
- [ ] Explain what `SettingWithCopyWarning` means and how to avoid it
- [ ] Save a DataFrame to CSV with `index=False`

---

[[04-basic-data-analysis|Previous: Basic Data Analysis]] | [[06-interview-questions|Next: Interview Questions]]
