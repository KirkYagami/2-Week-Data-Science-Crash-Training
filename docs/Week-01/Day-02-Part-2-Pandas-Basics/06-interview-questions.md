# 🎯 06 — Interview Questions: Pandas Basics

> These questions cover the Pandas fundamentals expected in beginner Data Science interviews.

---

## Series and DataFrames

**Q1:** What is Pandas?

> **Answer:** Pandas is a Python library for working with structured/tabular data. It provides two main objects: `Series` and `DataFrame`.

---

**Q2:** What is a Series?

> **Answer:** A `Series` is a one-dimensional labeled array. It is like a single column in a table.
> ```python
> import pandas as pd
>
> scores = pd.Series([85, 90, 78])
> ```

---

**Q3:** What is a DataFrame?

> **Answer:** A `DataFrame` is a two-dimensional labeled table made of rows and columns.
> ```python
> df = pd.DataFrame({
>     "name": ["Alice", "Bob"],
>     "score": [85, 90]
> })
> ```

---

**Q4:** What is the difference between a Series and a DataFrame?

> **Answer:** A `Series` is one-dimensional, like one column. A `DataFrame` is two-dimensional, like a full table with multiple columns.

---

**Q5:** How do you check the shape of a DataFrame?

> **Answer:**
> ```python
> df.shape
> ```
> It returns `(rows, columns)`.

---

**Q6:** What is the difference between `head()` and `tail()`?

> **Answer:** `head()` shows the first rows. `tail()` shows the last rows.
> ```python
> df.head()
> df.tail()
> df.head(10)
> ```

---

## Selecting Data

**Q7:** How do you select one column?

> **Answer:**
> ```python
> df["name"]
> ```
> This returns a `Series`.

---

**Q8:** How do you select multiple columns?

> **Answer:**
> ```python
> df[["name", "score"]]
> ```
> This returns a `DataFrame`.

---

**Q9:** What is the difference between `.loc` and `.iloc`?

> **Answer:** `.loc` selects by labels. `.iloc` selects by integer positions.
> ```python
> df.loc[0, "name"]
> df.iloc[0, 0]
> ```

---

**Q10:** How do you select rows based on a condition?

> **Answer:**
> ```python
> df[df["score"] >= 80]
> ```

---

**Q11:** How do you filter using multiple conditions?

> **Answer:** Use `&` for AND and `|` for OR. Put each condition in parentheses.
> ```python
> df[(df["score"] >= 80) & (df["city"] == "Delhi")]
> ```

---

**Q12:** Why can't you use `and` and `or` for Pandas filters?

> **Answer:** `and` and `or` expect single Boolean values, but Pandas conditions produce a Series of Boolean values. Use `&` and `|` instead.

---

**Q13:** What does `.isin()` do?

> **Answer:** `.isin()` checks whether each value exists in a list of allowed values.
> ```python
> df[df["city"].isin(["Delhi", "Mumbai"])]
> ```

---

## Reading and Writing Files

**Q14:** How do you read a CSV file in Pandas?

> **Answer:**
> ```python
> df = pd.read_csv("data.csv")
> ```

---

**Q15:** How do you read an Excel file?

> **Answer:**
> ```python
> df = pd.read_excel("data.xlsx")
> ```

---

**Q16:** How do you save a DataFrame to CSV?

> **Answer:**
> ```python
> df.to_csv("output.csv", index=False)
> ```

---

**Q17:** Why do we often use `index=False` when saving CSV files?

> **Answer:** It prevents Pandas from writing the DataFrame index as an extra column in the CSV file.

---

**Q18:** What does `usecols` do in `read_csv()`?

> **Answer:** It loads only selected columns.
> ```python
> df = pd.read_csv("data.csv", usecols=["name", "salary"])
> ```

---

## Sorting and Analysis

**Q19:** How do you sort a DataFrame by a column?

> **Answer:**
> ```python
> df.sort_values("salary")
> df.sort_values("salary", ascending=False)
> ```

---

**Q20:** How do you sort by multiple columns?

> **Answer:**
> ```python
> df.sort_values(["department", "salary"], ascending=[True, False])
> ```

---

**Q21:** How do you find the top 5 rows by a numeric column?

> **Answer:**
> ```python
> df.nlargest(5, "salary")
> ```

---

**Q22:** What does `describe()` do?

> **Answer:** `describe()` returns summary statistics for numeric columns, including count, mean, standard deviation, min, quartiles, and max.
> ```python
> df.describe()
> ```

---

**Q23:** What does `value_counts()` do?

> **Answer:** It counts how often each unique value appears in a column.
> ```python
> df["department"].value_counts()
> ```

---

**Q24:** How do you check missing values?

> **Answer:**
> ```python
> df.isna().sum()
> ```

---

**Q25:** How do you create a new column?

> **Answer:**
> ```python
> df["revenue"] = df["quantity"] * df["price"]
> ```

---

**Q26:** How do you rename columns?

> **Answer:**
> ```python
> df = df.rename(columns={"old_name": "new_name"})
> ```

---

**Q27:** How do you drop a column?

> **Answer:**
> ```python
> df = df.drop(columns=["column_name"])
> ```

---

## Beginner Scenario Questions

**Q28:** A CSV loads into one single column. What is likely wrong?

> **Answer:** The separator is probably not a comma. Use the `sep` parameter.
> ```python
> df = pd.read_csv("data.tsv", sep="\t")
> df = pd.read_csv("data_semicolon.csv", sep=";")
> ```

---

**Q29:** You see an `Unnamed: 0` column after reading a CSV. Why?

> **Answer:** The file was probably saved with the index included. Save future files with `index=False`, or read the existing file with `index_col=0`.

---

**Q30:** What first checks should you run after loading a dataset?

> **Answer:**
> ```python
> df.head()
> df.shape
> df.info()
> df.dtypes
> df.isna().sum()
> ```

---

**Q31:** How would you calculate total revenue from quantity and price columns?

> **Answer:**
> ```python
> df["revenue"] = df["quantity"] * df["price"]
> total_revenue = df["revenue"].sum()
> ```

---

**Q32:** How do you calculate total revenue by category?

> **Answer:**
> ```python
> df.groupby("category")["revenue"].sum()
> ```

---

## Quick Revision Checklist

- [ ] Explain Series vs DataFrame
- [ ] Create a DataFrame from a dictionary
- [ ] Read CSV and Excel files
- [ ] Select one column and multiple columns
- [ ] Use `.loc` and `.iloc`
- [ ] Filter with one condition
- [ ] Filter with multiple conditions
- [ ] Use `.isin()`
- [ ] Sort with `sort_values()`
- [ ] Use `describe()` and `value_counts()`
- [ ] Check missing values
- [ ] Create new calculated columns
- [ ] Save with `to_csv(index=False)`

---

## 🔗 What's Next?

➡️ [[../Day-03-Part-1-Pandas-Advanced/01-groupby|Day 03 — Pandas Advanced]]
