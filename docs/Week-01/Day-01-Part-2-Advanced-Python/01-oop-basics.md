# OOP Basics: Object-Oriented Programming

Every scikit-learn model you will ever call — `LinearRegression`, `RandomForestClassifier`, `StandardScaler` — is a class. When you write `model.fit(X_train, y_train)`, you are calling a method on an object. Understanding OOP is not optional for data science; it is the grammar of the tools you use every day.

**Prerequisites:** [[../Day-01-Part-1-Python-Basics/04-functions|Functions]]

## Learning Objectives

- Explain why classes exist and what problem they solve
- Build a class with `__init__`, instance methods, class attributes, and dunder methods
- Use inheritance and `super()` to extend existing classes without duplicating code
- Apply `@property`, `@classmethod`, and `@staticmethod` where each is appropriate
- Understand encapsulation and the single-underscore vs double-underscore conventions
- Recognize the OOP patterns that appear throughout scikit-learn

---

## Why Classes Exist

Before classes, the only way to group related data was a dictionary or a bunch of global variables. That works for small scripts. It falls apart when your codebase grows.

Imagine tracking a machine learning model's metadata without a class:

```python
# Without a class — everything is loose
model_name = "RandomForest"
model_version = "1.2"
model_accuracy = 0.94
model_features = ["age", "income", "tenure"]

# Now you have a second model. Do you do this?
model2_name = "XGBoost"
model2_version = "2.0"
# ...this scales terribly
```

A class lets you bundle the data and the operations on it into one coherent unit. Create as many instances as you need, each carrying its own state.

> [!info]
> OOP is not about architecture patterns or design philosophy at this stage. It is about organizing data and behavior so code stays manageable when it grows.

---

## Classes and Objects

A **class** is a blueprint. An **object** is a specific thing built from that blueprint.

```python
class ModelMetadata:
    """Tracks metadata for a trained ML model."""

    def __init__(self, name: str, version: str, accuracy: float):
        # __init__ runs when you create a new instance
        # self refers to the specific object being created
        self.name = name
        self.version = version
        self.accuracy = accuracy
        self.features: list = []

    def add_feature(self, feature_name: str) -> None:
        self.features.append(feature_name)

    def is_production_ready(self) -> bool:
        """Return True if model meets the accuracy bar for production."""
        return self.accuracy >= 0.90 and len(self.features) > 0

    def summary(self) -> str:
        return f"{self.name} v{self.version} — accuracy: {self.accuracy:.1%}"


# Create objects (instances)
rf_model = ModelMetadata("RandomForest", "1.2", 0.94)
xgb_model = ModelMetadata("XGBoost", "2.0", 0.87)

rf_model.add_feature("age")
rf_model.add_feature("income")
rf_model.add_feature("tenure")

print(rf_model.summary())           # Output: RandomForest v1.2 — accuracy: 94.0%
print(rf_model.is_production_ready())  # Output: True
print(xgb_model.is_production_ready()) # Output: False (accuracy below threshold)
```

> [!tip]
> Name classes with `PascalCase` (each word capitalized, no underscores). Name methods and attributes with `snake_case`. This is PEP 8 and everyone follows it.

---

## The `self` Parameter

`self` is how a method refers to the specific object it belongs to. It is always the first parameter of any instance method, and Python passes it automatically — you never write it when calling.

```python
class Counter:
    def __init__(self, start: int = 0):
        self.count = start

    def increment(self, step: int = 1) -> None:
        self.count += step  # modifies THIS instance's count

    def reset(self) -> None:
        self.count = 0


page_views = Counter()
api_calls = Counter(start=100)

page_views.increment()
page_views.increment()
api_calls.increment(step=5)

print(page_views.count)   # Output: 2
print(api_calls.count)    # Output: 105 — completely independent
```

When you call `page_views.increment()`, Python translates this to `Counter.increment(page_views)`. The name `self` is just a convention — but never deviate from it.

---

## `__str__` and `__repr__`

These two dunder methods control how your objects display. Get them right and debugging becomes much easier.

- `__str__`: user-facing string, called by `print()` and `str()`
- `__repr__`: developer-facing string, shown in the REPL and logs — should be unambiguous

```python
class DataPoint:
    def __init__(self, label: str, value: float, timestamp: str):
        self.label = label
        self.value = value
        self.timestamp = timestamp

    def __str__(self) -> str:
        # What a user sees
        return f"{self.label}: {self.value} (at {self.timestamp})"

    def __repr__(self) -> str:
        # What a developer sees — enough to recreate the object
        return f"DataPoint(label={self.label!r}, value={self.value}, timestamp={self.timestamp!r})"


reading = DataPoint("temperature", 36.6, "2025-01-15T14:30")

print(reading)        # Output: temperature: 36.6 (at 2025-01-15T14:30)
print(repr(reading))  # Output: DataPoint(label='temperature', value=36.6, timestamp='2025-01-15T14:30')

# In a list, Python uses __repr__
sensors = [reading, DataPoint("humidity", 72.1, "2025-01-15T14:30")]
print(sensors)  # uses __repr__ for each element
```

> [!warning]
> If you only define `__repr__`, Python uses it for both. If you only define `__str__`, `repr()` shows the unhelpful default `<__main__.DataPoint object at 0x...>`. Define both.

---

## Class Attributes vs Instance Attributes

An instance attribute belongs to one specific object. A class attribute belongs to the class itself and is shared across all instances.

```python
class APIClient:
    # Class attribute — shared by every APIClient instance
    base_url = "https://api.example.com"
    request_timeout = 30
    _instance_count = 0  # underscore = internal use

    def __init__(self, api_key: str, version: str = "v1"):
        # Instance attributes — unique to each object
        self.api_key = api_key
        self.version = version
        self._request_count = 0
        APIClient._instance_count += 1

    def get_endpoint(self, path: str) -> str:
        return f"{APIClient.base_url}/{self.version}/{path}"

    @classmethod
    def get_instance_count(cls) -> int:
        """Class method — operates on the class, not an instance."""
        return cls._instance_count

    @staticmethod
    def is_valid_api_key(key: str) -> bool:
        """Static method — no access to instance or class state."""
        return isinstance(key, str) and len(key) == 32


client_a = APIClient("aaabbbccc" + "d" * 23)
client_b = APIClient("zzzyyy" + "x" * 26, version="v2")

print(client_a.get_endpoint("users"))  # Output: https://api.example.com/v1/users
print(client_b.get_endpoint("users"))  # Output: https://api.example.com/v2/users
print(APIClient.get_instance_count())  # Output: 2
print(APIClient.is_valid_api_key("x" * 32))  # Output: True
```

Use `@classmethod` when you need access to the class (e.g., alternate constructors, factory methods). Use `@staticmethod` for utility functions that logically belong with the class but do not touch class or instance state.

---

## Inheritance

Inheritance lets a child class reuse everything from a parent class and add or change only what it needs.

```python
class BaseModel:
    """Parent class — defines the interface all models must follow."""

    def __init__(self, model_name: str, random_state: int = 42):
        self.model_name = model_name
        self.random_state = random_state
        self._is_fitted = False

    def fit(self, X: list, y: list) -> "BaseModel":
        raise NotImplementedError(f"{self.__class__.__name__} must implement fit()")

    def predict(self, X: list) -> list:
        if not self._is_fitted:
            raise RuntimeError("Call fit() before predict()")
        raise NotImplementedError

    def __repr__(self) -> str:
        fitted = "fitted" if self._is_fitted else "unfitted"
        return f"{self.__class__.__name__}(name={self.model_name!r}, {fitted})"


class MeanBaseline(BaseModel):
    """Predicts the mean of training labels — useful baseline."""

    def __init__(self, random_state: int = 42):
        # Call the parent __init__ using super()
        super().__init__(model_name="MeanBaseline", random_state=random_state)
        self._mean_value: float = 0.0

    def fit(self, X: list, y: list) -> "MeanBaseline":
        self._mean_value = sum(y) / len(y)
        self._is_fitted = True
        return self  # enables method chaining

    def predict(self, X: list) -> list:
        super().predict(X)  # triggers the fitted check
        return [self._mean_value] * len(X)


class WeightedMeanBaseline(MeanBaseline):
    """Baseline that weights recent samples more heavily."""

    def __init__(self, recency_weight: float = 0.7):
        super().__init__()
        self.model_name = "WeightedMeanBaseline"
        self.recency_weight = recency_weight

    def fit(self, X: list, y: list) -> "WeightedMeanBaseline":
        if len(y) == 0:
            raise ValueError("Cannot fit on empty labels")
        # Weight the second half of the data more
        midpoint = len(y) // 2
        early = y[:midpoint]
        recent = y[midpoint:]
        early_mean = sum(early) / len(early) if early else 0
        recent_mean = sum(recent) / len(recent) if recent else 0
        self._mean_value = (
            (1 - self.recency_weight) * early_mean
            + self.recency_weight * recent_mean
        )
        self._is_fitted = True
        return self


# Usage
train_labels = [10, 12, 11, 15, 18, 20, 22, 25]
test_features = [[1], [2], [3]]

baseline = MeanBaseline().fit(train_labels, train_labels)
weighted = WeightedMeanBaseline(recency_weight=0.8).fit(train_labels, train_labels)

print(baseline.predict(test_features))   # Output: [16.625, 16.625, 16.625]
print(weighted.predict(test_features))   # Output: [~21.0, ~21.0, ~21.0]
print(repr(baseline))  # Output: MeanBaseline(name='MeanBaseline', fitted)
```

> [!info]
> `super()` without arguments refers to the parent class. It is the correct way to call an overridden method from a child class. Always call `super().__init__()` first in a child's `__init__` unless you have a specific reason not to.

---

## The `@property` Decorator

Properties give you the syntax of attribute access with the control of a method. Use them when an attribute's value should be computed or validated on read or write.

```python
class Dataset:
    """Represents a tabular dataset with validation."""

    def __init__(self, name: str, records: list):
        self.name = name
        self._records = records
        self._target_column: str | None = None

    @property
    def record_count(self) -> int:
        """Read-only — always reflects current state."""
        return len(self._records)

    @property
    def target_column(self) -> str | None:
        return self._target_column

    @target_column.setter
    def target_column(self, column_name: str) -> None:
        if not self._records:
            raise ValueError("Cannot set target on empty dataset")
        available = list(self._records[0].keys()) if self._records else []
        if column_name not in available:
            raise ValueError(
                f"Column '{column_name}' not found. Available: {available}"
            )
        self._target_column = column_name

    def __len__(self) -> int:
        return self._records.__len__()

    def __repr__(self) -> str:
        return f"Dataset(name={self.name!r}, records={self.record_count})"


employee_data = [
    {"name": "Alice", "department": "Engineering", "salary": 95000},
    {"name": "Bob", "department": "Marketing", "salary": 72000},
    {"name": "Carol", "department": "Engineering", "salary": 88000},
]

ds = Dataset("employees", employee_data)

print(ds.record_count)  # Output: 3
print(len(ds))          # Output: 3

ds.target_column = "salary"
print(ds.target_column)  # Output: salary

try:
    ds.target_column = "nonexistent"
except ValueError as e:
    print(e)  # Output: Column 'nonexistent' not found. Available: ['name', 'department', 'salary']
```

---

## Dunder (Special) Methods

Dunder methods let your objects work with Python's built-in operators and functions. A well-designed class feels native to the language.

```python
class FeatureSet:
    """A collection of feature names for an ML model."""

    def __init__(self, features: list[str]):
        self._features = list(features)

    def __len__(self) -> int:
        return len(self._features)

    def __contains__(self, item: str) -> bool:
        """Enables: 'age' in feature_set"""
        return item in self._features

    def __iter__(self):
        """Enables: for feature in feature_set"""
        return iter(self._features)

    def __getitem__(self, index):
        """Enables: feature_set[0]"""
        return self._features[index]

    def __add__(self, other: "FeatureSet") -> "FeatureSet":
        """Enables: set_a + set_b"""
        combined = self._features + [f for f in other if f not in self._features]
        return FeatureSet(combined)

    def __repr__(self) -> str:
        return f"FeatureSet({self._features})"


numeric_features = FeatureSet(["age", "income", "tenure"])
categorical_features = FeatureSet(["city", "plan_type"])

all_features = numeric_features + categorical_features

print(len(all_features))                # Output: 5
print("age" in all_features)            # Output: True
print("unknown" in all_features)        # Output: False
print(all_features[0])                  # Output: age

for feature in all_features:
    print(feature)                      # Prints each feature name
```

| Dunder method | Triggered by |
|---|---|
| `__init__` | `ClassName(...)` |
| `__str__` | `print(obj)`, `str(obj)` |
| `__repr__` | `repr(obj)`, REPL display |
| `__len__` | `len(obj)` |
| `__contains__` | `item in obj` |
| `__iter__` | `for x in obj` |
| `__getitem__` | `obj[key]` |
| `__add__` | `obj + other` |
| `__eq__` | `obj == other` |
| `__lt__` | `obj < other` |

---

## Dataclasses — When You Need a Class for Data Storage

Python 3.7 added `@dataclass` to reduce boilerplate for classes that primarily store data.

```python
from dataclasses import dataclass, field
from typing import Optional


@dataclass
class TrainingConfig:
    model_name: str
    learning_rate: float = 0.001
    max_epochs: int = 100
    batch_size: int = 32
    random_state: int = 42
    feature_columns: list = field(default_factory=list)
    target_column: Optional[str] = None

    def is_valid(self) -> bool:
        return (
            self.learning_rate > 0
            and self.max_epochs > 0
            and self.batch_size > 0
        )


# __init__, __repr__, and __eq__ are generated automatically
config = TrainingConfig(
    model_name="XGBoost",
    learning_rate=0.05,
    max_epochs=200,
    feature_columns=["age", "income", "tenure"],
    target_column="churn",
)

print(config)
# Output: TrainingConfig(model_name='XGBoost', learning_rate=0.05, max_epochs=200, batch_size=32, ...)

print(config.is_valid())  # Output: True

# __eq__ works out of the box
config2 = TrainingConfig(model_name="XGBoost", learning_rate=0.05, max_epochs=200,
                         feature_columns=["age", "income", "tenure"], target_column="churn")
print(config == config2)  # Output: True
```

> [!tip]
> Use `@dataclass` when your class is primarily a data container with few or no complex methods. Use a regular class when behavior is central to what the class does.

---

## Putting It Together — A Data Science Example

Here is the OOP pattern you will see over and over in production pipelines:

```python
from dataclasses import dataclass, field


@dataclass
class ValidationResult:
    is_valid: bool
    errors: list = field(default_factory=list)

    def __str__(self) -> str:
        if self.is_valid:
            return "ValidationResult: PASSED"
        return f"ValidationResult: FAILED — {'; '.join(self.errors)}"


class RecordValidator:
    """Validates employee records before inserting into a database."""

    REQUIRED_FIELDS = {"name", "age", "department", "salary"}

    def __init__(self, min_salary: float = 0, max_age: int = 100):
        self.min_salary = min_salary
        self.max_age = max_age
        self._validated_count = 0
        self._failed_count = 0

    def validate(self, record: dict) -> ValidationResult:
        errors = []

        missing = self.REQUIRED_FIELDS - record.keys()
        if missing:
            errors.append(f"Missing fields: {sorted(missing)}")

        if "age" in record:
            age = record["age"]
            if not isinstance(age, int) or not (0 <= age <= self.max_age):
                errors.append(f"age must be int in 0–{self.max_age}, got {age!r}")

        if "salary" in record:
            salary = record["salary"]
            if not isinstance(salary, (int, float)) or salary < self.min_salary:
                errors.append(f"salary must be >= {self.min_salary}, got {salary!r}")

        if "name" in record:
            if not isinstance(record["name"], str) or not record["name"].strip():
                errors.append("name must be a non-empty string")

        result = ValidationResult(is_valid=len(errors) == 0, errors=errors)
        if result.is_valid:
            self._validated_count += 1
        else:
            self._failed_count += 1
        return result

    @property
    def stats(self) -> dict:
        total = self._validated_count + self._failed_count
        return {
            "total": total,
            "passed": self._validated_count,
            "failed": self._failed_count,
            "pass_rate": self._validated_count / total if total > 0 else 0,
        }

    def __repr__(self) -> str:
        return (
            f"RecordValidator(min_salary={self.min_salary}, max_age={self.max_age})"
        )


# Usage
validator = RecordValidator(min_salary=20000)

records = [
    {"name": "Alice", "age": 31, "department": "Engineering", "salary": 95000},
    {"name": "", "age": 31, "department": "Engineering", "salary": 95000},
    {"name": "Bob", "age": 150, "department": "Marketing", "salary": 72000},
    {"name": "Carol", "department": "Sales", "salary": 60000},  # missing age
]

for rec in records:
    result = validator.validate(rec)
    print(result)

print(validator.stats)
# Output: {'total': 4, 'passed': 1, 'failed': 3, 'pass_rate': 0.25}
```

---

## Key Takeaways

> [!success]
> - A **class** is a blueprint; an **object** is an instance of that blueprint
> - `__init__` initializes an object's state when it is created
> - `self` refers to the specific object a method is operating on
> - **Inheritance** (`class Child(Parent)`) reuses and extends behavior — always call `super().__init__()`
> - `@property` gives you controlled attribute access without breaking the calling interface
> - `@classmethod` operates on the class itself; `@staticmethod` is a function scoped to the class
> - **Dunder methods** make your objects feel native to Python (`len()`, `in`, `+`, etc.)
> - `@dataclass` cuts boilerplate for data-holding classes

---

[[00-agenda|← Agenda]] | [[02-file-handling|File Handling →]]
