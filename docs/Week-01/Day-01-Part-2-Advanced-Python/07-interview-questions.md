# 🎯 07 – Interview Questions: Advanced Python

> These questions test whether you can write Python that is clean, reusable, safe, and suitable for real Data Science work.

---

## 🏗️ Object-Oriented Programming

**Q1:** What is a class in Python?

> **Answer:** A class is a blueprint for creating objects. It groups related data and behavior together.
> ```python
> class Student:
>     def __init__(self, name, score):
>         self.name = name
>         self.score = score
>
>     def is_passing(self):
>         return self.score >= 60
>
>
> student = Student("Alice", 85)
> print(student.is_passing())  # True
> ```

---

**Q2:** What is the difference between a class and an object?

> **Answer:** A class is the definition. An object is a specific instance created from that class.
> ```python
> class ModelConfig:
>     pass
>
> config = ModelConfig()  # object / instance
> ```

---

**Q3:** What is `self`?

> **Answer:** `self` refers to the current object. It lets methods access and modify the object's attributes.
> ```python
> class Counter:
>     def __init__(self):
>         self.value = 0
>
>     def increment(self):
>         self.value += 1
> ```
> `self.value` belongs to each individual `Counter` object.

---

**Q4:** What is the purpose of `__init__`?

> **Answer:** `__init__` is the constructor method. It runs automatically when a new object is created and is usually used to initialize attributes.
> ```python
> class Dataset:
>     def __init__(self, path):
>         self.path = path
> ```

---

**Q5:** What is inheritance?

> **Answer:** Inheritance allows one class to reuse or extend another class.
> ```python
> class Model:
>     def train(self):
>         print("Training model")
>
>
> class LinearRegression(Model):
>     def predict(self, x):
>         return x
> ```
> `LinearRegression` inherits the `train()` method from `Model`.

---

**Q6:** What is method overriding?

> **Answer:** Method overriding happens when a child class defines a method with the same name as a parent class method.
> ```python
> class Model:
>     def predict(self):
>         return "generic prediction"
>
>
> class Classifier(Model):
>     def predict(self):
>         return "class label"
> ```

---

## 📁 File Handling

**Q7:** Why should you use `with open(...)` when working with files?

> **Answer:** `with open(...)` automatically closes the file, even if an error occurs.
> ```python
> with open("data.txt", "r") as file:
>     content = file.read()
> ```
> This is safer than manually calling `file.close()`.

---

**Q8:** What is the difference between read mode, write mode, and append mode?

> **Answer:**
> | Mode | Meaning |
> |------|---------|
> | `"r"` | Read an existing file |
> | `"w"` | Write a file, replacing existing content |
> | `"a"` | Append to the end of a file |
> | `"x"` | Create a new file, fail if it already exists |

---

**Q9:** How do you read a CSV file as dictionaries?

> **Answer:** Use `csv.DictReader`.
> ```python
> import csv
>
> with open("students.csv", "r", newline="") as file:
>     reader = csv.DictReader(file)
>     for row in reader:
>         print(row["name"], row["score"])
> ```

---

**Q10:** How do you read and write JSON files?

> **Answer:** Use the built-in `json` module.
> ```python
> import json
>
> with open("config.json", "r") as file:
>     config = json.load(file)
>
> with open("output.json", "w") as file:
>     json.dump(config, file, indent=2)
> ```

---

## 📦 Modules and Packages

**Q11:** What is a module in Python?

> **Answer:** A module is a Python file that contains reusable code. For example, `data_utils.py` can be imported into another file.
> ```python
> # data_utils.py
> def mean(values):
>     return sum(values) / len(values)
> ```
> ```python
> # main.py
> from data_utils import mean
>
> print(mean([1, 2, 3]))
> ```

---

**Q12:** What is the difference between a module and a package?

> **Answer:** A module is a single `.py` file. A package is a folder containing multiple modules, often with an `__init__.py` file.
>
> ```text
> analytics/
>   __init__.py
>   cleaning.py
>   metrics.py
> ```

---

**Q13:** What is the difference between `import module` and `from module import function`?

> **Answer:**
> ```python
> import math
> print(math.sqrt(16))
> ```
> This imports the module name.
>
> ```python
> from math import sqrt
> print(sqrt(16))
> ```
> This imports the function directly.

---

**Q14:** Why should you avoid `from module import *`?

> **Answer:** It pollutes the namespace and makes it unclear where names came from. This can cause bugs when two modules define the same name.

---

**Q15:** What is the purpose of `if __name__ == "__main__"`?

> **Answer:** It lets a file behave differently when run directly versus imported as a module.
> ```python
> def main():
>     print("Running script")
>
>
> if __name__ == "__main__":
>     main()
> ```
> When imported, `main()` will not run automatically.

---

## ⚠️ Exception Handling

**Q16:** What is the purpose of exception handling?

> **Answer:** Exception handling lets your program recover from expected errors instead of crashing immediately.
> ```python
> try:
>     age = int("abc")
> except ValueError:
>     age = None
> ```

---

**Q17:** What is the difference between `except Exception` and a bare `except:`?

> **Answer:** A bare `except:` catches almost everything, including system-level signals you usually should not catch. `except Exception` is safer, but catching specific exceptions like `ValueError` or `FileNotFoundError` is best.

---

**Q18:** What is the difference between `try`, `except`, `else`, and `finally`?

> **Answer:**
> | Block | Runs When |
> |-------|-----------|
> | `try` | Code that might fail |
> | `except` | An error is caught |
> | `else` | No error occurred |
> | `finally` | Always runs |
>
> ```python
> try:
>     value = int("10")
> except ValueError:
>     print("Invalid number")
> else:
>     print("Success")
> finally:
>     print("Done")
> ```

---

**Q19:** When should you raise an exception?

> **Answer:** Raise an exception when a function cannot safely continue or receives invalid input.
> ```python
> def calculate_average(values):
>     if not values:
>         raise ValueError("values cannot be empty")
>     return sum(values) / len(values)
> ```

---

**Q20:** What is a custom exception?

> **Answer:** A custom exception is an error class you define for your own domain.
> ```python
> class DataValidationError(Exception):
>     pass
>
>
> def validate_age(age):
>     if age < 0:
>         raise DataValidationError("age cannot be negative")
> ```
> Custom exceptions make error handling clearer in larger projects.

---

## ✨ Python Best Practices

**Q21:** What is PEP 8?

> **Answer:** PEP 8 is the official Python style guide. It covers naming, spacing, indentation, imports, line length, and other readability standards.

---

**Q22:** What does "Pythonic" code mean?

> **Answer:** Pythonic code uses Python's language features naturally and clearly.
> ```python
> # Not Pythonic
> squares = []
> for number in range(5):
>     squares.append(number * number)
>
> # Pythonic
> squares = [number**2 for number in range(5)]
> ```

---

**Q23:** Why are meaningful variable names important?

> **Answer:** Good names make code easier to read, debug, and maintain.
> ```python
> # Weak
> x = [85, 90, 78]
>
> # Better
> exam_scores = [85, 90, 78]
> ```

---

**Q24:** Why should functions usually do one thing?

> **Answer:** Small focused functions are easier to test, reuse, debug, and explain in interviews.
> ```python
> def clean_name(name):
>     return name.strip().title()
>
>
> def is_valid_age(age):
>     return isinstance(age, int) and 0 <= age <= 120
> ```

---

**Q25:** What is the mutable default argument problem?

> **Answer:** Default arguments are created once when the function is defined, not each time it is called.
> ```python
> # Bug
> def add_item(item, items=[]):
>     items.append(item)
>     return items
>
> print(add_item("a"))  # ['a']
> print(add_item("b"))  # ['a', 'b']
> ```
>
> Use `None` instead:
> ```python
> def add_item(item, items=None):
>     if items is None:
>         items = []
>     items.append(item)
>     return items
> ```

---

**Q26:** What is the difference between shallow copy and deep copy?

> **Answer:** A shallow copy copies the outer object only. A deep copy also copies nested objects.
> ```python
> import copy
>
> original = [[1, 2], [3, 4]]
> shallow = copy.copy(original)
> deep = copy.deepcopy(original)
> ```
> For nested lists, dictionaries, or configs, deep copy prevents accidental changes to shared inner objects.

---

**Q27:** Why are docstrings useful?

> **Answer:** Docstrings explain what a function, class, or module does. They are especially useful when code becomes reusable.
> ```python
> def calculate_rmse(actual, predicted):
>     """Return root mean squared error for two equal-length sequences."""
>     ...
> ```

---

## 🧠 Data Science Scenario Questions

**Q28:** You receive a CSV with some invalid numeric values. What should your Python code do?

> **Answer:** It should parse values safely, log or report bad rows, skip or repair invalid records based on the business rule, and avoid crashing on the first bad value.
> ```python
> try:
>     salary = float(row["salary"])
> except (KeyError, ValueError, TypeError):
>     salary = None
> ```

---

**Q29:** Why might you create a class for a machine learning pipeline step?

> **Answer:** A class can store configuration and expose reusable methods such as `fit()`, `transform()`, and `predict()`. This is the pattern used by scikit-learn.
> ```python
> class MissingValueFiller:
>     def __init__(self, fill_value=0):
>         self.fill_value = fill_value
>
>     def transform(self, values):
>         return [self.fill_value if value is None else value for value in values]
> ```

---

**Q30:** How would you organize reusable data cleaning functions?

> **Answer:** Put them in a separate module, use clear function names, write small tests or examples, and import them wherever needed.
>
> ```text
> project/
>   main.py
>   data_cleaning.py
>   validation.py
> ```

---

## ✅ Quick Revision Checklist

- [ ] Explain class vs object
- [ ] Explain `self` and `__init__`
- [ ] Use `with open(...)` for files
- [ ] Read CSV with `csv.DictReader`
- [ ] Read/write JSON with `json.load()` and `json.dump()`
- [ ] Explain module vs package
- [ ] Use `if __name__ == "__main__"`
- [ ] Catch specific exceptions
- [ ] Explain `try/except/else/finally`
- [ ] Write Pythonic list comprehensions
- [ ] Explain mutable default arguments
- [ ] Connect OOP to Data Science pipelines

---

## 🔗 What's Next?

➡️ Day 02 — Start using NumPy and Pandas for real tabular data analysis
