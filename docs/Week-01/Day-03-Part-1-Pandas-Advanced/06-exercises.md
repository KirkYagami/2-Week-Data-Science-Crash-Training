# Exercises — Pandas Advanced

These exercises build on everything from Day 3: groupby, merge, apply functions, missing values, and data cleaning. Work through each one before expanding the solution. You learn more from five minutes of struggling than from five seconds of reading an answer.

## Setup

```python
import pandas as pd
import numpy as np
```

---

## Exercise 1 — GroupBy: Sales Summary

**Dataset:**

```python
sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106, 107, 108, 109, 110],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories",
                 "Accessories", "Electronics", "Electronics", "Accessories",
                 "Electronics", "Accessories"],
    "product": ["Laptop", "Mouse", "Monitor", "Keyboard", "Mouse", "Laptop",
                "Monitor", "Keyboard", "Laptop", "Mouse"],
    "quantity": [2, 5, 1, 3, 10, 1, 2, 4, 3, 6],
    "price": [75000, 600, 15000, 1200, 600, 75000, 15000, 1200, 75000, 600],
    "sales_rep": ["Priya", "Rahul", "Priya", "Anjali", "Rahul", "Anjali",
                  "Priya", "Rahul", "Priya", "Anjali"]
})

sales["revenue"] = sales["quantity"] * sales["price"]
```

**Warm-up:**
1. What is the total revenue by city?
2. How many orders did each sales rep handle?

**Main:**

3. Build a named aggregation summary by category with these columns: `total_revenue`, `avg_order_value`, `total_quantity`, `order_count`.
4. Add a column `city_total_revenue` that shows the total revenue for each order's city on every row (use `.transform()`).
5. Add a column `pct_of_city_revenue` showing each order's share of its city's total revenue, rounded to 1 decimal.

**Stretch:**

6. Find the top-selling product (by revenue) within each city.
7. Find all orders where the order revenue is above the category average. Return only `order_id`, `category`, `revenue`, and a new column `category_avg`.

??? "Show solution"

    ```python
    # Warm-up 1: total revenue by city
    city_rev = sales.groupby("city", as_index=False)["revenue"].sum()
    print(city_rev)
    # Output:
    #      city  revenue
    # 0   Delhi   420000
    # 1  Mumbai    12000
    # 2    Pune    49200

    # Warm-up 2: order count by rep
    print(sales.groupby("sales_rep")["order_id"].count())
    # Output:
    # sales_rep
    # Anjali    3
    # Priya     4
    # Rahul     3

    # Main 3: named aggregation summary by category
    summary = sales.groupby("category").agg(
        total_revenue=("revenue", "sum"),
        avg_order_value=("revenue", "mean"),
        total_quantity=("quantity", "sum"),
        order_count=("order_id", "count")
    )
    print(summary)

    # Main 4: city total revenue on every row using transform
    sales["city_total_revenue"] = sales.groupby("city")["revenue"].transform("sum")

    # Main 5: percentage of city revenue
    sales["pct_of_city_revenue"] = (
        sales["revenue"] / sales["city_total_revenue"] * 100
    ).round(1)
    print(sales[["order_id", "city", "revenue", "city_total_revenue", "pct_of_city_revenue"]])

    # Stretch 6: top product by revenue within each city
    def top_product(group):
        return group.nlargest(1, "revenue")[["city", "product", "revenue"]]

    top_by_city = sales.groupby("city").apply(top_product).reset_index(drop=True)
    print(top_by_city)

    # Stretch 7: orders above category average
    sales["category_avg"] = sales.groupby("category")["revenue"].transform("mean")
    above_avg = sales[sales["revenue"] > sales["category_avg"]]
    print(above_avg[["order_id", "category", "revenue", "category_avg"]])
    ```

---

## Exercise 2 — Merge: Build a Full Order Report

**Datasets:**

```python
orders = pd.DataFrame({
    "order_id": [201, 202, 203, 204, 205, 206],
    "customer_id": [1, 2, 1, 3, 4, 9],    # customer 9 does not exist
    "product_id": [10, 11, 12, 10, 13, 11],  # product 13 does not exist
    "quantity": [2, 1, 3, 1, 2, 5],
    "order_date": pd.to_datetime(["2024-01-10", "2024-01-12", "2024-01-15",
                                  "2024-01-18", "2024-01-20", "2024-01-22"])
})

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5],   # customer 5 has no orders
    "customer_name": ["Alice Sharma", "Bob Menon", "Charlie Rao", "Diana Nair", "Evan Patel"],
    "city": ["Delhi", "Mumbai", "Pune", "Bangalore", "Delhi"],
    "tier": ["Gold", "Silver", "Gold", "Bronze", "Silver"]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],   # product 13 is missing
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200],
    "category": ["Electronics", "Accessories", "Accessories"]
})
```

**Warm-up:**
1. Merge orders with customers using a left join. How many orders have no customer match?

**Main:**

2. Build a full order table by chaining merges with customers and products. Use `validate="many_to_one"` on both.
3. Calculate `revenue` for each order (`quantity * price`).
4. Which orders have missing product information? Show their `order_id` and `product_id`.

**Stretch:**

5. Calculate total revenue by city and by customer tier. Handle missing customer rows appropriately.
6. Find the customer who generated the most revenue. Include their tier.

??? "Show solution"

    ```python
    # Warm-up 1
    left_joined = pd.merge(orders, customers, on="customer_id", how="left")
    no_customer_match = left_joined["customer_name"].isna().sum()
    print(f"Orders with no customer match: {no_customer_match}")
    # Output: Orders with no customer match: 1  (customer_id=9)

    # Main 2: full order table with validation
    full_orders = (
        orders
        .merge(customers, on="customer_id", how="left", validate="many_to_one")
        .merge(products, on="product_id", how="left", validate="many_to_one")
    )

    # Main 3: revenue
    full_orders["revenue"] = full_orders["quantity"] * full_orders["price"]
    print(full_orders[["order_id", "customer_name", "product_name", "revenue"]])

    # Main 4: orders with missing product info
    missing_product = full_orders[full_orders["product_name"].isna()]
    print(missing_product[["order_id", "product_id"]])
    # Output:
    #    order_id  product_id
    # 4       205          13

    # Stretch 5: revenue by city and tier
    # Drop rows with missing customer info for fair aggregation
    matched = full_orders.dropna(subset=["customer_name", "product_name"])

    print("Revenue by city:")
    print(matched.groupby("city")["revenue"].sum().sort_values(ascending=False))

    print("\nRevenue by customer tier:")
    print(matched.groupby("tier")["revenue"].sum().sort_values(ascending=False))

    # Stretch 6: top customer by revenue
    customer_rev = (
        matched.groupby(["customer_name", "tier"])["revenue"]
        .sum()
        .reset_index()
        .sort_values("revenue", ascending=False)
    )
    print(customer_rev.head(1))
    ```

---

## Exercise 3 — Apply Functions: Employee Transformations

**Dataset:**

```python
employees = pd.DataFrame({
    "emp_id": [1001, 1002, 1003, 1004, 1005, 1006, 1007],
    "name": [" alice sharma ", "BOB MENON", "Charlie Rao", " diana nair", "EVAN PATEL", " fatima khan", "GOPAL SINGH "],
    "department": ["Sales", "Engineering", "Engineering", "HR", "Sales", "Engineering", "HR"],
    "salary": [52000, 95000, 88000, 44000, 61000, 110000, 39000],
    "experience_years": [2, 7, 5, 1, 3, 9, 0],
    "performance_rating": [4.1, 4.9, 4.4, 3.7, 4.2, 4.8, 3.5],
    "region": ["North", "South", "South", "North", "West", "South", "North"]
})
```

**Warm-up:**
1. Clean the `name` column: strip whitespace and apply title case.
2. Map departments to codes: `Engineering → ENG`, `Sales → SLS`, `HR → HRM`.

**Main:**

3. Create a `salary_band` column using `np.select()`:
   - Band 4 (Principal): salary >= 90,000
   - Band 3 (Senior): salary >= 65,000
   - Band 2 (Mid): salary >= 45,000
   - Band 1 (Junior): below 45,000

4. Calculate annual bonus using vectorized operations (no `apply`):
   - Base bonus: 10% of salary
   - Top performer bonus (rating >= 4.7): add ₹15,000
   - Seniority bonus (experience >= 7 years): add ₹8,000

5. Create a `years_bucket` column using `pd.cut()` with bins: `[0, 2, 5, 10]` and labels `["Junior", "Mid", "Senior"]`.

**Stretch:**

6. For each department, compute the ratio of each employee's salary to the department maximum salary. Call it `dept_salary_ratio`. Do this without `apply`.
7. Write a function `flag_high_value_employee(row)` that returns `True` if the employee is in the top band AND has experience >= 5 years AND rating >= 4.5. Apply it with `apply(axis=1)`. Then rewrite it using vectorized operations and verify the results match.

??? "Show solution"

    ```python
    # Warm-up 1: clean names
    employees["name"] = employees["name"].str.strip().str.title()

    # Warm-up 2: department codes
    dept_map = {"Engineering": "ENG", "Sales": "SLS", "HR": "HRM"}
    employees["dept_code"] = employees["department"].map(dept_map)
    # Verify no NaN (all depts mapped)
    assert employees["dept_code"].isna().sum() == 0

    # Main 3: salary bands with np.select
    conditions = [
        employees["salary"] >= 90000,
        employees["salary"] >= 65000,
        employees["salary"] >= 45000
    ]
    labels = ["Band 4 (Principal)", "Band 3 (Senior)", "Band 2 (Mid)"]
    employees["salary_band"] = np.select(conditions, labels, default="Band 1 (Junior)")

    # Main 4: bonus — fully vectorized
    base_bonus = employees["salary"] * 0.10
    top_performer = np.where(employees["performance_rating"] >= 4.7, 15000, 0)
    seniority = np.where(employees["experience_years"] >= 7, 8000, 0)
    employees["annual_bonus"] = base_bonus + top_performer + seniority
    print(employees[["name", "salary", "performance_rating", "experience_years", "annual_bonus"]])

    # Main 5: years bucket
    employees["years_bucket"] = pd.cut(
        employees["experience_years"],
        bins=[0, 2, 5, 10],
        labels=["Junior", "Mid", "Senior"],
        include_lowest=True
    )

    # Stretch 6: salary ratio to department max — using transform
    employees["dept_max_salary"] = employees.groupby("department")["salary"].transform("max")
    employees["dept_salary_ratio"] = (employees["salary"] / employees["dept_max_salary"]).round(3)
    print(employees[["name", "department", "salary", "dept_max_salary", "dept_salary_ratio"]])

    # Stretch 7: high value employee — apply version
    def flag_high_value_employee(row):
        return (
            row["salary"] >= 90000
            and row["experience_years"] >= 5
            and row["performance_rating"] >= 4.5
        )

    employees["high_value_apply"] = employees.apply(flag_high_value_employee, axis=1)

    # Vectorized version — same result
    employees["high_value_vec"] = (
        (employees["salary"] >= 90000)
        & (employees["experience_years"] >= 5)
        & (employees["performance_rating"] >= 4.5)
    )

    # Verify they match
    assert (employees["high_value_apply"] == employees["high_value_vec"]).all()
    print("Both methods match.")
    print(employees[["name", "salary_band", "experience_years", "performance_rating", "high_value_vec"]])
    ```

---

## Exercise 4 — Missing Values: Customer Data Audit

**Dataset:**

```python
customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5, 6, 7, 8],
    "name": ["Alice", "Bob", None, "Diana", "Evan", None, "Gopal", "Hema"],
    "age": [25, np.nan, 32, 29, np.nan, 41, np.nan, 35],
    "city": ["Delhi", "Mumbai", None, "Pune", "Delhi", "Mumbai", None, "Pune"],
    "annual_spend": [12000, 25000, np.nan, 18000, 0, np.nan, 8000, np.nan],
    "plan": ["Premium", "Basic", "Premium", np.nan, "Basic", "Premium", np.nan, "Basic"]
})
```

**Warm-up:**
1. Print the missing count and missing percentage for each column.
2. Show all rows where `city` is missing.

**Main:**

3. Create missingness indicator columns for `age`, `annual_spend`, and `plan` before doing any filling.
4. Fill missing `age` with the median age by city (use `groupby` + `transform`). Fill any remaining NaN with the global median.
5. Fill missing `annual_spend` with the median spend by `plan` tier. Fill remaining NaN with the global median.
6. Fill missing `city` with `"Unknown"`. Fill missing `plan` with `"No Plan"`. Drop rows where `name` is missing.

**Stretch:**

7. After cleaning, verify: no NaN in `customer_id`, `name`, `age`, `city`, or `annual_spend`.
8. Build a reusable `audit_missing(df)` function that returns a DataFrame with `missing_count`, `missing_pct`, and `dtype` for each column, sorted by missing percentage descending.

??? "Show solution"

    ```python
    # Warm-up 1: missing audit
    print(customers.isna().sum())
    print((customers.isna().mean() * 100).round(1))

    # Warm-up 2: rows where city is missing
    print(customers[customers["city"].isna()])

    # Main 3: missingness indicators BEFORE filling
    customers["age_was_missing"] = customers["age"].isna()
    customers["spend_was_missing"] = customers["annual_spend"].isna()
    customers["plan_was_missing"] = customers["plan"].isna()

    # Main 4: group-based age fill, then global fallback
    customers["age"] = customers["age"].fillna(
        customers.groupby("city")["age"].transform("median")
    )
    customers["age"] = customers["age"].fillna(customers["age"].median())

    # Main 5: group-based spend fill, then global fallback
    customers["annual_spend"] = customers["annual_spend"].fillna(
        customers.groupby("plan")["annual_spend"].transform("median")
    )
    customers["annual_spend"] = customers["annual_spend"].fillna(
        customers["annual_spend"].median()
    )

    # Main 6: fill categoricals, drop missing name
    customers["city"] = customers["city"].fillna("Unknown")
    customers["plan"] = customers["plan"].fillna("No Plan")
    customers = customers.dropna(subset=["name"])

    print(customers)

    # Stretch 7: assertions
    critical_cols = ["customer_id", "name", "age", "city", "annual_spend"]
    for col in critical_cols:
        missing = customers[col].isna().sum()
        assert missing == 0, f"Column '{col}' still has {missing} missing values"
    print("All critical columns are complete.")

    # Stretch 8: reusable audit function
    def audit_missing(df: pd.DataFrame) -> pd.DataFrame:
        total = len(df)
        return pd.DataFrame({
            "missing_count": df.isna().sum(),
            "missing_pct": (df.isna().mean() * 100).round(1),
            "dtype": df.dtypes
        }).sort_values("missing_pct", ascending=False)

    print(audit_missing(customers))
    ```

---

## Exercise 5 — Real-World Cleaning Pipeline

**Dataset:**

```python
raw = pd.DataFrame({
    " Order ID ": [501, 502, 503, 503, 504, 505],
    "Customer Name": [" ALICE SHARMA ", "Bob Menon", " charlie rao", "charlie rao", None, "FATIMA KHAN"],
    "Product": ["Laptop", "mouse", "KEYBOARD", "KEYBOARD", "Monitor", "laptop"],
    "Qty": ["2", "five", "3", "3", "1", "2"],
    "Unit Price": ["75,000", "600", "1,200", "1,200", "-500", "N/A"],
    "Order Date": ["2024-01-10", "2024/01/12", "15-01-2024", "15-01-2024", "not a date", "2024-01-20"],
    "City": ["delhi", "MUMBAI", "Pune ", " pune", "BANGALORE", "mumbai"]
})
```

**Warm-up:**
1. Standardize column names to lowercase with underscores.
2. Inspect the data: print shape, dtypes, and the raw values of each column.

**Main:**

3. Remove duplicate order IDs (keep first occurrence).
4. Replace `"N/A"`, `"five"`, and `"not a date"` with NaN.
5. Clean: title-case `customer_name`, `product`, `city`. Strip whitespace everywhere.
6. Convert `qty` to integer and `unit_price` to float (removing commas first). Use `errors="coerce"`.
7. Convert `order_date` to datetime. Extract year and month into separate columns.
8. Fix impossible values: set negative prices to NaN.
9. Fill missing prices with the median. Drop rows with missing `customer_name`.
10. Add a `revenue` column (`qty * unit_price`).

**Stretch:**

11. Wrap steps 1–10 in a single `clean_orders(raw)` function that returns the cleaned DataFrame.
12. Add a validation step inside the function that raises a `ValueError` if any of these are true: `order_id` has duplicates, `qty` has negative values, `unit_price` has negative values.

??? "Show solution"

    ```python
    import pandas as pd
    import numpy as np


    def clean_orders(raw: pd.DataFrame) -> pd.DataFrame:
        df = raw.copy()

        # Step 1: standardize column names
        df.columns = (
            df.columns.str.strip().str.lower()
            .str.replace(" ", "_", regex=False)
            .str.replace(r"[^a-z0-9_]", "", regex=True)
        )

        # Step 2: replace placeholders
        df = df.replace(["N/A", "n/a", "not a date", "five", "", "-"], np.nan)

        # Step 3: deduplicate on order_id
        df["order_id"] = pd.to_numeric(df["order_id"], errors="coerce")
        df = df.drop_duplicates(subset=["order_id"], keep="first")

        # Step 4: clean text columns
        for col in ["customer_name", "product", "city"]:
            df[col] = df[col].str.strip().str.title()

        # Step 5: convert numeric columns
        df["qty"] = pd.to_numeric(df["qty"], errors="coerce")
        df["unit_price"] = (
            df["unit_price"]
            .str.replace(",", "", regex=False)
            .pipe(pd.to_numeric, errors="coerce")
        )

        # Step 6: convert dates
        df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
        df["order_year"] = df["order_date"].dt.year
        df["order_month"] = df["order_date"].dt.month

        # Step 7: fix impossible values
        df.loc[df["unit_price"] < 0, "unit_price"] = np.nan
        df.loc[df["qty"] < 0, "qty"] = np.nan

        # Step 8: handle missing values
        df["unit_price"] = df["unit_price"].fillna(df["unit_price"].median())
        df["qty"] = df["qty"].fillna(df["qty"].median())
        df = df.dropna(subset=["customer_name"])

        # Step 9: revenue
        df["revenue"] = df["qty"] * df["unit_price"]

        # Validation
        errors = []
        if df["order_id"].duplicated().any():
            errors.append(f"order_id has duplicates: {df['order_id'].duplicated().sum()}")
        if (df["qty"].dropna() < 0).any():
            errors.append("qty has negative values")
        if (df["unit_price"].dropna() < 0).any():
            errors.append("unit_price has negative values")
        if errors:
            raise ValueError("Validation failed:\n" + "\n".join(f"  - {e}" for e in errors))

        return df


    cleaned = clean_orders(raw)
    print(cleaned)
    print("\nColumn types:")
    print(cleaned.dtypes)
    print("\nMissing values:")
    print(cleaned.isna().sum())
    ```

---

## Final Challenge — End-to-End Customer Revenue Report

This challenge combines everything from the day: cleaning, merging, transformations, and grouped analysis.

**Datasets:**

```python
orders_raw = pd.DataFrame({
    "order_id": [601, 602, 603, 604, 605, 606, 607],
    "customer_id": [1, 2, 1, 3, 4, 9, 2],    # customer 9 is an orphan
    "product_id": [10, 11, 12, 10, 99, 11, 12],  # product 99 does not exist
    "quantity": [2, 1, 3, 1, 2, 5, 1],
    "order_date": ["2024-Q1", "2024-01-12", "2024-01-15",
                   "bad date", "2024-02-20", "2024-02-22", "2024-03-01"]
})

customers_raw = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "customer_name": [" ALICE SHARMA ", "Bob Menon", None, "Diana Nair"],
    "city": ["delhi", "MUMBAI", "Pune", ""],
    "tier": ["Gold", "SILVER", "Gold", "Bronze"]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200],
    "category": ["Electronics", "Accessories", "Accessories"]
})
```

**Tasks:**

1. Clean `customers_raw`: standardize name, city, tier. Fill missing name with `"Unknown"` and empty city with `"Unknown"`.
2. Clean `orders_raw`: convert `order_date` to datetime with `errors="coerce"`.
3. Merge orders with cleaned customers (left join, validate many-to-one).
4. Merge result with products (left join, validate many-to-one).
5. Identify and print orders with no customer match and orders with no product match.
6. Calculate `revenue` for matched orders only (where price is not NaN).
7. Build a report: total revenue and order count by city and by tier.
8. Add a column showing each order's percentage of total revenue (across all matched orders).
9. Find which product category generated the most revenue.

??? "Show solution"

    ```python
    import pandas as pd
    import numpy as np

    # Step 1: clean customers
    customers = customers_raw.copy()
    customers["customer_name"] = customers["customer_name"].str.strip().str.title()
    customers["city"] = customers["city"].str.strip().str.title()
    customers["tier"] = customers["tier"].str.strip().str.title()
    customers = customers.replace({"": np.nan, "None": np.nan})
    customers["customer_name"] = customers["customer_name"].fillna("Unknown")
    customers["city"] = customers["city"].fillna("Unknown")

    # Step 2: clean orders
    orders = orders_raw.copy()
    orders["order_id"] = pd.to_numeric(orders["order_id"], errors="coerce")
    orders["customer_id"] = pd.to_numeric(orders["customer_id"], errors="coerce")
    orders["product_id"] = pd.to_numeric(orders["product_id"], errors="coerce")
    orders["quantity"] = pd.to_numeric(orders["quantity"], errors="coerce")
    orders["order_date"] = pd.to_datetime(orders["order_date"], errors="coerce")

    # Step 3: merge with customers
    report = orders.merge(customers, on="customer_id", how="left", validate="many_to_one")

    # Step 4: merge with products
    report = report.merge(products, on="product_id", how="left", validate="many_to_one")

    # Step 5: identify unmatched rows
    no_customer = report[report["customer_name"].isna() | (report["customer_name"] == "Unknown")]
    no_product = report[report["product_name"].isna()]
    print(f"Orders with no customer match: {(report['tier'].isna()).sum()}")
    print(no_product[["order_id", "product_id"]])

    # Step 6: revenue for matched orders
    matched = report.dropna(subset=["price"])
    matched = matched.copy()
    matched["revenue"] = matched["quantity"] * matched["price"]

    # Step 7: summary by city and tier
    print("\nRevenue by city:")
    city_summary = matched.groupby("city").agg(
        total_revenue=("revenue", "sum"),
        order_count=("order_id", "count")
    ).sort_values("total_revenue", ascending=False)
    print(city_summary)

    print("\nRevenue by tier:")
    tier_summary = matched.groupby("tier").agg(
        total_revenue=("revenue", "sum"),
        order_count=("order_id", "count")
    ).sort_values("total_revenue", ascending=False)
    print(tier_summary)

    # Step 8: percentage of total revenue per order
    total_revenue = matched["revenue"].sum()
    matched["pct_of_total"] = (matched["revenue"] / total_revenue * 100).round(2)
    print("\nRevenue share by order:")
    print(matched[["order_id", "customer_name", "product_name", "revenue", "pct_of_total"]])

    # Step 9: top product category
    category_rev = matched.groupby("category")["revenue"].sum().sort_values(ascending=False)
    print(f"\nTop revenue category: {category_rev.index[0]} (₹{category_rev.iloc[0]:,.0f})")
    ```

---

## Self-Check

You are ready to move on if you can:

- [ ] Explain the split-apply-combine model and use `.groupby().agg()` with named aggregations
- [ ] Use `.transform()` to attach group-level statistics to original rows without merging
- [ ] Choose the correct join type and use `validate=` to catch duplicate-key bugs
- [ ] Identify when to use `pd.merge()` vs `pd.concat()` vs `.join()`
- [ ] Describe the vectorization performance hierarchy and know when `.apply()` is appropriate
- [ ] Use `.map()`, `.str` accessors, `.dt` accessors, and `np.select()` in the right situations
- [ ] Audit missing data and distinguish between NaN and invalid placeholder values
- [ ] Choose between group-based filling, global median, forward fill, and constant fill
- [ ] Execute a cleaning pipeline in the correct order and validate the output

---

[[05-real-world-data-cleaning]] — From messy table to analysis-ready dataset | [[../Day-03-Part-2-Data-Visualization/01-matplotlib-basics|Data Visualization]] — Visualize your clean data
