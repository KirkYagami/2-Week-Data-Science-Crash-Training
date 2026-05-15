# Interview Questions: Advanced Python

These questions appear in technical screenings for data science, data engineering, and ML engineering roles. Work through them in order. Read each question, think through your answer, then reveal the model answer. The goal is not to memorize — it is to build fluency so you can explain clearly under pressure.

**Prerequisites:** [[01-oop-basics]], [[02-file-handling]], [[03-modules-and-packages]], [[04-exception-handling]], [[05-python-best-practices]]

---

## Object-Oriented Programming

**Q1. What is the difference between a class and an object?**

??? "Show answer"
    A class is a blueprint — it defines structure and behavior. An object is a specific instance created from that blueprint. You can create many objects from one class, each with its own independent state.

    ```python
    class ModelConfig:
        def __init__(self, model_type: str, learning_rate: float):
            self.model_type = model_type
            self.learning_rate = learning_rate

    # Two objects from one class — independent state
    rf_config = ModelConfig("random_forest", 0.01)
    xgb_config = ModelConfig("xgboost", 0.05)

    print(rf_config.model_type)   # Output: random_forest
    print(xgb_config.model_type)  # Output: xgboost
    ```

    The analogy: the class is the cookie cutter, each object is a cookie. The cookies are separate and can be decorated differently, but they came from the same template.

---

**Q2. What does `self` refer to, and why is it always the first parameter of an instance method?**

??? "Show answer"
    `self` refers to the specific object the method is being called on. When you write `alice.is_passing()`, Python translates this to `Student.is_passing(alice)` — `alice` is automatically passed as `self`. This is how each instance has its own independent state.

    ```python
    class Counter:
        def __init__(self, start: int = 0):
            self.count = start  # self.count belongs to THIS specific counter

        def increment(self) -> None:
            self.count += 1

    page_views = Counter()
    api_calls = Counter(start=100)

    page_views.increment()
    page_views.increment()

    print(page_views.count)  # Output: 2
    print(api_calls.count)   # Output: 100 — completely independent
    ```

    The name `self` is a convention, not a keyword. You could name it anything. Do not — everyone uses `self`.

---

**Q3. What is `__init__`, and how is it different from a constructor in other languages?**

??? "Show answer"
    `__init__` is Python's initializer — it runs immediately after an object is created and sets up its initial state. It is commonly called a constructor, though technically the object already exists when `__init__` runs (the real constructor is `__new__`, which you rarely need to touch).

    ```python
    class Dataset:
        def __init__(self, name: str, records: list):
            # This runs when you write: Dataset("customers", [...])
            self.name = name
            self._records = records
            self._is_validated = False  # always starts unvalidated

        def validate(self) -> bool:
            self._is_validated = True
            return True

    ds = Dataset("customers", [{"id": 1, "name": "Alice"}])
    print(ds.name)           # Output: customers
    print(ds._is_validated)  # Output: False
    ds.validate()
    print(ds._is_validated)  # Output: True
    ```

    In Python you do not need `new` — `Dataset(...)` does everything.

---

**Q4. Explain the difference between a class attribute and an instance attribute.**

??? "Show answer"
    A class attribute is defined directly on the class and shared by all instances. An instance attribute is defined in `__init__` (or other methods) with `self.` and is unique to each object.

    ```python
    class Employee:
        company_name = "DataCorp"  # class attribute — same for everyone
        employee_count = 0

        def __init__(self, name: str, salary: float):
            self.name = name        # instance attribute — unique per employee
            self.salary = salary
            Employee.employee_count += 1  # modify the class attribute

    alice = Employee("Alice", 95000)
    bob = Employee("Bob", 72000)

    print(alice.company_name)      # Output: DataCorp  (class attribute)
    print(alice.name)              # Output: Alice  (instance attribute)
    print(bob.name)                # Output: Bob
    print(Employee.employee_count) # Output: 2
    ```

    A critical trap: if you do `alice.company_name = "NewCorp"`, Python creates a new instance attribute on `alice` that shadows the class attribute. `bob.company_name` still returns `"DataCorp"`. Modify class attributes via the class, not an instance: `Employee.company_name = "NewCorp"`.

---

**Q5. What is inheritance? Why use it?**

??? "Show answer"
    Inheritance lets a child class reuse everything from a parent class and add or override only what differs. Use it to model "is-a" relationships — a `RandomForest` is a `Model`, a `Manager` is an `Employee`.

    ```python
    class BaseModel:
        def __init__(self, name: str):
            self.name = name
            self._is_fitted = False

        def fit(self, X, y):
            raise NotImplementedError(f"{self.__class__.__name__} must implement fit()")

        def __repr__(self) -> str:
            status = "fitted" if self._is_fitted else "unfitted"
            return f"{self.name}({status})"


    class MeanBaseline(BaseModel):
        def __init__(self):
            super().__init__("MeanBaseline")  # call parent's __init__
            self._mean = 0.0

        def fit(self, X: list, y: list) -> "MeanBaseline":
            self._mean = sum(y) / len(y)
            self._is_fitted = True
            return self

        def predict(self, X: list) -> list:
            return [self._mean] * len(X)


    model = MeanBaseline().fit([1, 2, 3], [10, 20, 30])
    print(repr(model))            # Output: MeanBaseline(fitted)
    print(model.predict([0, 0]))  # Output: [20.0, 20.0]
    ```

    Do not use inheritance for code reuse alone — that often leads to deep inheritance chains that are hard to follow. Prefer composition (has-a) when in doubt. Scikit-learn uses inheritance for its estimator interface because every model genuinely is-a BaseEstimator.

---

**Q6. What does `super()` do and when should you use it?**

??? "Show answer"
    `super()` returns a proxy to the parent class. The most common use is calling the parent's `__init__` from a child's `__init__` to avoid duplicating initialization logic.

    ```python
    class Shape:
        def __init__(self, color: str):
            self.color = color
            self.created_at = "2025-01-15"

        def describe(self) -> str:
            return f"A {self.color} shape"


    class Circle(Shape):
        def __init__(self, color: str, radius: float):
            super().__init__(color)  # runs Shape.__init__, sets self.color and self.created_at
            self.radius = radius     # adds Circle-specific attribute

        def area(self) -> float:
            import math
            return math.pi * self.radius ** 2


    c = Circle("red", 5.0)
    print(c.color)        # Output: red  (set by Shape.__init__ via super())
    print(c.radius)       # Output: 5.0
    print(c.describe())   # Output: A red shape  (inherited method)
    print(f"{c.area():.2f}")  # Output: 78.54
    ```

    If you forget `super().__init__()`, the parent's initialization never runs — attributes set by the parent will not exist on the child object.

---

**Q7. What is a `@property` and when would you use it over a plain attribute?**

??? "Show answer"
    `@property` lets you access a method as if it were an attribute — without parentheses. Use it when:

    - The value should be computed from other attributes
    - You want to make an attribute read-only
    - You want to validate a value when it is set

    ```python
    class SalesRecord:
        def __init__(self, units_sold: int, unit_price: float):
            self.units_sold = units_sold
            self._unit_price = unit_price

        @property
        def revenue(self) -> float:
            """Computed on every access — always up to date."""
            return self.units_sold * self._unit_price

        @property
        def unit_price(self) -> float:
            return self._unit_price

        @unit_price.setter
        def unit_price(self, value: float) -> None:
            if value < 0:
                raise ValueError(f"unit_price cannot be negative, got {value}")
            self._unit_price = value


    sale = SalesRecord(units_sold=10, unit_price=1500.0)
    print(sale.revenue)        # Output: 15000.0  (no parentheses!)

    sale.units_sold = 15
    print(sale.revenue)        # Output: 22500.0  (automatically updated)

    try:
        sale.unit_price = -100
    except ValueError as e:
        print(e)               # Output: unit_price cannot be negative, got -100
    ```

    If you ever need to add validation to an existing attribute, changing it to a `@property` does not break any calling code — callers still write `sale.unit_price = value`, they never know it is now a method.

---

**Q8. What is the difference between `@classmethod` and `@staticmethod`?**

??? "Show answer"
    Both are methods that can be called on the class without creating an instance. The difference is what they receive:

    - `@classmethod` receives the class as its first argument (`cls`). Use for alternate constructors or operations that need class-level state.
    - `@staticmethod` receives nothing special — it is a plain function scoped to the class for organizational reasons.

    ```python
    class FeatureSet:
        def __init__(self, features: list[str], source: str = "manual"):
            self.features = features
            self.source = source

        @classmethod
        def from_csv_header(cls, header_line: str) -> "FeatureSet":
            """Alternate constructor — create from a CSV header row."""
            features = [col.strip() for col in header_line.split(",")]
            return cls(features, source="csv")

        @classmethod
        def from_dict_keys(cls, sample_record: dict) -> "FeatureSet":
            """Alternate constructor — create from a sample record."""
            return cls(list(sample_record.keys()), source="dict")

        @staticmethod
        def is_valid_feature_name(name: str) -> bool:
            """Check if a string is a valid feature name — no class state needed."""
            return isinstance(name, str) and name.replace("_", "").isalnum()

        def __repr__(self) -> str:
            return f"FeatureSet({self.features}, source={self.source!r})"


    fs1 = FeatureSet.from_csv_header("age, income, tenure, churn")
    fs2 = FeatureSet.from_dict_keys({"age": 31, "income": 72000})

    print(fs1)  # Output: FeatureSet(['age', 'income', 'tenure', 'churn'], source='csv')
    print(fs2)  # Output: FeatureSet(['age', 'income'], source='dict')
    print(FeatureSet.is_valid_feature_name("monthly_charges"))  # Output: True
    print(FeatureSet.is_valid_feature_name("2bad"))             # Output: False
    ```

---

## File Handling

**Q9. Why should you always use `with open()` instead of `open()` + `close()`?**

??? "Show answer"
    `with open()` uses Python's context manager protocol, which guarantees the file is closed when the `with` block exits — even if an exception is raised inside the block. Manual `open()` + `close()` will leak a file handle if anything raises before `close()` is reached.

    ```python
    # Dangerous — exception between open() and close() leaks the handle
    f = open("data.csv", "r")
    content = f.read()
    process(content)  # if this raises, f.close() never runs
    f.close()

    # Safe — the context manager closes the file regardless
    with open("data.csv", "r", encoding="utf-8") as f:
        content = f.read()
        process(content)  # exception here? File still gets closed.
    ```

    File descriptor leaks are silent. On a long-running server, they eventually cause the process to stop being able to open any file. Always use `with`.

---

**Q10. You receive a CSV file and some values fail to parse. How do you handle it without crashing the entire pipeline?**

??? "Show answer"
    Parse each row inside `try / except`, log the failure, and continue. The pipeline processes all valid rows and reports what was skipped.

    ```python
    import csv

    def load_and_parse(filepath: str) -> tuple[list[dict], list[dict]]:
        """Return (valid_records, error_records)."""
        valid_records = []
        error_records = []

        with open(filepath, "r", newline="", encoding="utf-8") as f:
            for row_num, row in enumerate(csv.DictReader(f), start=2):
                try:
                    record = {
                        "customer_id": int(row["customer_id"]),
                        "age": int(row["age"]),
                        "monthly_charges": float(row["monthly_charges"]),
                    }
                    valid_records.append(record)
                except (KeyError, ValueError, TypeError) as e:
                    error_records.append({"row": row_num, "reason": str(e), "raw": dict(row)})

        return valid_records, error_records


    valid, errors = load_and_parse("customers.csv")
    print(f"Loaded {len(valid)} valid rows, skipped {len(errors)} rows")
    for err in errors:
        print(f"  Row {err['row']}: {err['reason']}")
    ```

    The key principle: one bad row should not stop the pipeline. Log it and move on.

---

**Q11. What does `newline=""` do in `open()` when working with CSV files on Windows?**

??? "Show answer"
    Without `newline=""`, Python's universal newline handling translates `\r\n` to `\n` on read, and on write it translates `\n` to `\r\n`. The `csv` module then adds its own newline handling. The interaction produces double newlines between rows on Windows — every row is followed by a blank row.

    Passing `newline=""` tells Python not to do any newline translation, leaving it entirely to the `csv` module.

    ```python
    # Correct on all platforms
    with open("data.csv", "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["name", "score"])
        writer.writeheader()
        writer.writerows([{"name": "Alice", "score": 92}])
    ```

    Always include `newline=""` when using the `csv` module.

---

## Modules and Packages

**Q12. What is the difference between `import math` and `from math import sqrt`?**

??? "Show answer"
    Both give you access to `sqrt`, but they differ in namespace:

    - `import math` imports the module object. You access its contents with `math.sqrt()`. The name `math` is added to your local namespace.
    - `from math import sqrt` imports the function directly into your local namespace. You call it as `sqrt()` without the prefix.

    ```python
    import math
    print(math.sqrt(144))     # Output: 12.0
    # math is now a name in local scope

    from math import sqrt, pi
    print(sqrt(144))           # Output: 12.0
    print(pi)                  # Output: 3.141592653589793
    # math is NOT in local scope — only sqrt and pi are
    ```

    For large libraries with many submodules (like `matplotlib.pyplot`), the `from x import y` form avoids long prefixes. For disambiguation when two modules export the same name, `import module` with dotted access is safer.

---

**Q13. What does `if __name__ == "__main__":` do and why is it important?**

??? "Show answer"
    Every Python module has a `__name__` attribute. When Python runs a file directly (e.g., `python my_script.py`), `__name__` is set to `"__main__"`. When it is imported by another file, `__name__` is set to the module's filename (e.g., `"my_script"`).

    This guard prevents code from running when the file is imported as a module:

    ```python
    # data_utils.py

    def normalize(values: list) -> list:
        min_v, max_v = min(values), max(values)
        if min_v == max_v:
            return [0.5] * len(values)
        return [(v - min_v) / (max_v - min_v) for v in values]


    if __name__ == "__main__":
        # Only runs when you execute: python data_utils.py
        # Does NOT run when: import data_utils
        test = [10, 20, 30, 40, 50]
        print(normalize(test))  # Output: [0.0, 0.25, 0.5, 0.75, 1.0]
    ```

    Without this guard, every `import data_utils` anywhere in your project would run the test code — printing to the console, potentially reading files, or triggering side effects you did not intend.

---

**Q14. Why should you avoid `from module import *`?**

??? "Show answer"
    It imports every name from the module into your current namespace without making the source explicit. This causes two problems:

    1. **Name collisions**: if two modules both define `mean`, the second import silently overwrites the first.
    2. **Unreadability**: a reader cannot tell where `mean`, `normalize`, or `clean_data` came from without searching.

    ```python
    # Opaque — where does clean_data come from?
    from data_utils import *
    from stats import *

    result = clean_data(records)   # data_utils.clean_data? stats.clean_data?

    # Clear — no ambiguity
    from data_utils import clean_data
    from stats import compute_mean

    result = clean_data(records)
    ```

    The only acceptable use of `import *` is in interactive sessions where you are exploring a library quickly and do not care about namespace hygiene.

---

## Exception Handling

**Q15. What is the difference between a bare `except:` and `except Exception:`?**

??? "Show answer"
    A bare `except:` catches absolutely everything, including `SystemExit` (raised by `sys.exit()`), `KeyboardInterrupt` (raised by Ctrl+C), and `GeneratorExit`. These are signals that should almost never be swallowed — they mean the user or the system wants the program to stop.

    `except Exception:` catches everything that inherits from `Exception`, which excludes `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit`.

    ```python
    # Worst — catches Ctrl+C, sys.exit(), everything
    try:
        run_training()
    except:
        pass  # user can't even kill this with Ctrl+C

    # Better but still too broad — hides real bugs
    try:
        run_training()
    except Exception as e:
        print(f"Something went wrong: {e}")

    # Best — catch what you expect and handle it appropriately
    try:
        run_training()
    except FileNotFoundError:
        print("Training data not found")
    except ValueError as e:
        print(f"Invalid configuration: {e}")
    ```

---

**Q16. Explain `try / except / else / finally` and what each block is for.**

??? "Show answer"
    ```python
    import json

    def load_config(path: str) -> dict | None:
        try:
            # The risky code — anything here might raise
            with open(path, "r") as f:
                config = json.load(f)

        except FileNotFoundError:
            # Runs if the specific exception was raised
            print(f"Config not found: {path}")
            return None

        except json.JSONDecodeError as e:
            print(f"Invalid JSON: {e}")
            return None

        else:
            # Runs ONLY if no exception occurred in try
            # Put code that should only execute on success here
            print(f"Loaded {len(config)} configuration keys")
            return config

        finally:
            # ALWAYS runs — even if there is a return in try or except
            # Use for cleanup: closing connections, flushing logs, releasing locks
            print(f"Finished processing {path}")
    ```

    | Block | Runs when |
    |-------|-----------|
    | `try` | Always |
    | `except` | Only when the matching exception was raised |
    | `else` | Only when no exception was raised |
    | `finally` | Always — even through a `return` statement |

    The `else` block exists so that success-path code does not accidentally get wrapped in the exception handler. If the return statement is inside `try` instead of `else`, it is technically inside the exception-handling scope, which can cause subtle issues.

---

**Q17. When should you raise an exception rather than returning `None` or a default value?**

??? "Show answer"
    Return `None` or a default when the failure is expected and the caller can meaningfully continue. Raise an exception when the failure means the caller cannot safely continue without knowing about it.

    ```python
    # Returning None is appropriate — caller can check
    def find_customer(customer_id: int, records: list) -> dict | None:
        for record in records:
            if record["id"] == customer_id:
                return record
        return None  # not found is a normal case


    # Raising is appropriate — missing required column is a programming error
    def extract_column(records: list, column: str) -> list:
        if not records:
            raise ValueError("Cannot extract column from empty records list")
        if column not in records[0]:
            available = list(records[0].keys())
            raise KeyError(
                f"Column '{column}' not found. Available: {available}"
            )
        return [r[column] for r in records]
    ```

    A good rule: if the failure is something the caller probably should handle (e.g., user input error, network timeout, missing optional data), return a sentinel. If the failure means something is wrong with the code itself (e.g., wrong types, violated contract), raise.

---

**Q18. What are custom exception classes and why are they useful?**

??? "Show answer"
    Custom exceptions let you name specific failure modes in your domain. Callers can catch exactly the failure they know how to handle, and let others propagate.

    ```python
    class PipelineError(Exception):
        """Base class for all pipeline errors."""
        pass

    class MissingColumnError(PipelineError):
        def __init__(self, column: str, available: list):
            self.column = column
            self.available = available
            super().__init__(
                f"Required column '{column}' not found. Available: {available}"
            )

    class DataQualityError(PipelineError):
        def __init__(self, column: str, message: str):
            self.column = column
            super().__init__(f"Data quality failure in '{column}': {message}")


    def validate(records: list, required: list) -> None:
        available = list(records[0].keys()) if records else []
        for col in required:
            if col not in available:
                raise MissingColumnError(col, available)


    try:
        validate(records, required=["age", "income", "churn_flag"])
    except MissingColumnError as e:
        print(f"Schema mismatch: {e}")
        # Handle schema errors specifically
    except DataQualityError as e:
        print(f"Bad data: {e}")
        # Handle data issues differently
    except PipelineError as e:
        print(f"Pipeline failed: {e}")
        # Catch any other pipeline error as a fallback
    ```

    The hierarchy lets callers be as specific or as general as they need. Generic `ValueError` does not give callers any information about what kind of error happened.

---

## Python Best Practices

**Q19. What is the mutable default argument problem? Show the bug and the fix.**

??? "Show answer"
    Default argument values are evaluated **once** when the function is defined, not each time it is called. If the default is a mutable object (list, dict, set), all calls that use the default share the same object.

    ```python
    # Bug — the same list is reused across all calls
    def append_feature(name: str, feature_list: list = []) -> list:
        feature_list.append(name)
        return feature_list

    result1 = append_feature("age")
    result2 = append_feature("income")

    print(result1)  # Output: ['age', 'income']  — wrong! expected ['age']
    print(result2)  # Output: ['age', 'income']  — same object!


    # Fix — use None as the sentinel
    def append_feature(name: str, feature_list: list | None = None) -> list:
        if feature_list is None:
            feature_list = []  # new list created on each call
        feature_list.append(name)
        return feature_list

    result1 = append_feature("age")
    result2 = append_feature("income")

    print(result1)  # Output: ['age']
    print(result2)  # Output: ['income']
    ```

    This is one of the most common Python gotchas. The fix is always the same: use `None` as the default and create the mutable object inside the function.

---

**Q20. What is a generator and when would you use one over a list?**

??? "Show answer"
    A generator is a function that uses `yield` to produce values one at a time. It does not build the entire sequence in memory — it computes each value on demand when the caller asks for the next one.

    Use a generator when:
    - The dataset is large (millions of rows)
    - You only need to iterate once
    - Memory is a constraint

    ```python
    # List — loads everything into memory at once
    def load_all(filepath: str) -> list[dict]:
        import csv
        with open(filepath, "r", newline="") as f:
            return list(csv.DictReader(f))
    # 10 million rows? That's gigabytes of RAM.


    # Generator — one row in memory at a time
    def stream_rows(filepath: str):
        import csv
        with open(filepath, "r", newline="") as f:
            reader = csv.DictReader(f)
            for row in reader:
                yield row  # produces one row, pauses here
    # Memory usage is constant regardless of file size.


    # Use it
    total_revenue = 0
    for row in stream_rows("sales.csv"):
        total_revenue += int(row["units_sold"]) * float(row["unit_price"])

    print(f"Total revenue: {total_revenue:,}")
    ```

    Generator expressions work like list comprehensions but lazily:
    ```python
    # Materializes all values: [0, 1, 4, 9, ...]
    squares = [x**2 for x in range(1_000_000)]

    # Lazy — only computes each value when needed
    squares = (x**2 for x in range(1_000_000))
    total = sum(squares)  # sum consumes the generator without building the list
    ```

---

**Q21. Explain the EAFP vs LBYL distinction. Which does Python prefer?**

??? "Show answer"
    **LBYL (Look Before You Leap)**: Check conditions before acting.

    ```python
    # LBYL
    import os
    if os.path.exists("data.csv") and os.path.isfile("data.csv"):
        with open("data.csv") as f:
            data = f.read()
    ```

    **EAFP (Easier to Ask Forgiveness than Permission)**: Try the action, handle the failure.

    ```python
    # EAFP
    try:
        with open("data.csv") as f:
            data = f.read()
    except FileNotFoundError:
        data = None
    ```

    Python culture and the language itself favor EAFP for two reasons:

    1. **Race conditions**: the file could be deleted between the `os.path.exists()` check and the `open()` call. EAFP handles the actual failure.
    2. **Clarity**: EAFP often has fewer lines and handles the case more directly.

    LBYL makes sense for simple pre-checks: `if denominator != 0:` before dividing. Use EAFP when the failure mode is an external resource (file, network, database) or user-supplied input.

---

**Q22. A colleague's data pipeline crashes in production with a `KeyError`. What are the three most likely causes and how do you investigate?**

??? "Show answer"
    A `KeyError` means code tried to access a dictionary key that does not exist. In a data pipeline, the three most common causes are:

    **1. The upstream data schema changed** — a column was renamed, removed, or added. The pipeline expected `"customer_id"` but the CSV now has `"cust_id"`.

    ```python
    # Investigation: print the available keys before accessing
    print(list(row.keys()))
    # or check against expected schema
    expected = {"customer_id", "age", "salary"}
    actual = set(row.keys())
    missing = expected - actual
    if missing:
        raise MissingColumnError(missing, actual)
    ```

    **2. Optional fields treated as required** — code accesses `row["phone_number"]` but not all records have a phone number. Use `.get()` to handle optional fields safely.

    ```python
    phone = row.get("phone_number")       # returns None if missing
    phone = row.get("phone_number", "")   # returns "" if missing
    ```

    **3. Environment difference** — the pipeline works with a sample file in development but fails with real production data because the production file has a different structure, encoding, or dialect.

    **Investigation approach**: add the key to the exception message so you know exactly which field is missing, and log the full raw row so you can see what was actually in the data.

    ```python
    try:
        salary = float(row["salary"])
    except KeyError:
        raise KeyError(f"'salary' missing from row. Available keys: {list(row.keys())}")
    ```

---

## Data Science Integration

**Q23. Why do scikit-learn models use classes with `fit()` and `transform()` methods rather than standalone functions?**

??? "Show answer"
    A scikit-learn estimator needs to remember state between fitting and transforming. A function cannot do that — a class can.

    For example, `StandardScaler` must learn the mean and standard deviation during `fit()` and then apply those same values during `transform()`. If these were separate functions, you would have to pass the learned parameters around manually.

    ```python
    class StandardScaler:
        def __init__(self):
            self._mean = None
            self._std = None

        def fit(self, X: list[float]) -> "StandardScaler":
            """Learn the mean and std from training data."""
            n = len(X)
            self._mean = sum(X) / n
            variance = sum((x - self._mean) ** 2 for x in X) / n
            self._std = variance ** 0.5
            return self

        def transform(self, X: list[float]) -> list[float]:
            """Apply the learned scaling to new data."""
            if self._mean is None:
                raise RuntimeError("Call fit() before transform()")
            return [(x - self._mean) / self._std for x in X]


    scaler = StandardScaler()
    scaler.fit([10, 20, 30, 40, 50])

    # Apply same scaling to train AND test — prevents data leakage
    train_scaled = scaler.transform([10, 20, 30, 40, 50])
    test_scaled = scaler.transform([15, 25])

    print([f"{v:.2f}" for v in train_scaled])  # Output: ['-1.41', '-0.71', '0.00', '0.71', '1.41']
    print([f"{v:.2f}" for v in test_scaled])    # Output: ['-1.06', '-0.35']
    ```

    The consistent `fit() / transform() / predict()` interface also enables sklearn's `Pipeline` to chain arbitrary steps together without knowing what each step does internally.

---

## Quick Revision Checklist

Before your next technical interview, make sure you can:

- [ ] Explain class vs object with an example from data science
- [ ] Write `__init__`, `__repr__`, and `__str__` from scratch
- [ ] Explain `self` in one sentence
- [ ] Use `super().__init__()` correctly in a child class
- [ ] Write a `@property` with a getter and setter
- [ ] Distinguish `@classmethod` from `@staticmethod`
- [ ] Use `with open()` with correct encoding and mode for CSV and JSON
- [ ] Explain why `newline=""` is required in the csv module
- [ ] Import a function from a module you created yourself
- [ ] Explain `if __name__ == "__main__":` and what happens without it
- [ ] Catch specific exceptions and explain why bare `except:` is dangerous
- [ ] Demonstrate the mutable default argument bug and fix
- [ ] Write a generator function and explain its memory benefit
- [ ] Apply EAFP correctly on a file operation

---

[[06-mini-exercises|← Mini Exercises]] | [[../../Day-02-Part-1-NumPy-Fundamentals/00-agenda|Day 02: NumPy →]]
