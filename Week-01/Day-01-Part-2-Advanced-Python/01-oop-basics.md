# 🏗️ 01 – OOP Basics: Object-Oriented Programming

> **Prerequisites:** [[../Day-01-Part-1-Python-Basics/04-functions|Functions]]  
> **Time to read:** ~35 minutes

---

## 🧠 What is Object-Oriented Programming?

**Object-Oriented Programming (OOP)** is a way of structuring programs by grouping related **data** (attributes) and **behavior** (methods) into objects.

### Real-World Analogy

Think of a **Car**:
- **Attributes** (data): color, brand, speed, fuel_level
- **Methods** (behavior): accelerate(), brake(), refuel()

In code:
```python
class Car:
    def __init__(self, brand, color):
        self.brand = brand
        self.color = color
        self.speed = 0

    def accelerate(self, amount):
        self.speed += amount

    def brake(self, amount):
        self.speed = max(0, self.speed - amount)
```

### Why OOP Matters for Data Science

| Scenario | OOP Benefit |
|----------|-------------|
| Custom ML pipeline steps | Each step is a class with `fit()` and `transform()` |
| Data validator | Class that holds rules and validates records |
| API client | Class that manages authentication and requests |
| Feature store | Class that loads, caches, and serves features |

Scikit-learn itself is built entirely on OOP — every model is a class.

---

## 📐 Classes and Objects

### The Blueprint Analogy

```
Class → Blueprint (definition)
Object → Instance (actual thing built from blueprint)

Class:  Car (defines what all cars are and can do)
Object: my_honda = Car("Honda", "Red")
        your_toyota = Car("Toyota", "Blue")
```

### Defining a Class

```python
class Student:
    """Represents a student in a course."""

    # Class attribute — shared by ALL instances
    school_name = "Data Science Academy"

    def __init__(self, name, age, score):
        """Initialize a new Student. __init__ is the constructor."""
        # Instance attributes — unique to each instance
        self.name = name
        self.age = age
        self.score = score

    def is_passing(self):
        """Return True if the student is passing (score >= 60)."""
        return self.score >= 60

    def get_grade(self):
        """Return the letter grade based on score."""
        if self.score >= 90:
            return "A"
        elif self.score >= 80:
            return "B"
        elif self.score >= 70:
            return "C"
        elif self.score >= 60:
            return "D"
        else:
            return "F"

    def __str__(self):
        """String representation — used by print()."""
        return f"Student({self.name}, age={self.age}, score={self.score})"

    def __repr__(self):
        """Detailed representation — used in debugging."""
        return f"Student(name={self.name!r}, age={self.age}, score={self.score})"
```

### Creating Objects (Instances)

```python
# Create instances
alice = Student("Alice", 20, 92)
bob = Student("Bob", 22, 75)
charlie = Student("Charlie", 21, 45)

# Access attributes
print(alice.name)             # Alice
print(bob.score)              # 75

# Call methods
print(alice.is_passing())     # True
print(charlie.is_passing())   # False
print(alice.get_grade())      # A
print(bob.get_grade())        # C

# Class attribute (same for all)
print(alice.school_name)      # Data Science Academy
print(Student.school_name)    # Data Science Academy

# __str__ is called by print()
print(alice)                  # Student(Alice, age=20, score=92)

# Check type
print(type(alice))            # <class '__main__.Student'>
print(isinstance(alice, Student))  # True
```

---

## 🔒 The `self` Parameter

`self` refers to the **current instance**. When you call `alice.is_passing()`, Python automatically passes `alice` as `self`.

```python
class Counter:
    def __init__(self, start=0):
        self.value = start   # self.value is an instance attribute

    def increment(self, by=1):
        self.value += by     # modifies THIS instance's value

    def reset(self):
        self.value = 0

c1 = Counter(10)
c2 = Counter(0)

c1.increment(5)
print(c1.value)   # 15
print(c2.value)   # 0  — unchanged! separate instance
```

---

## 🧩 Class vs Instance Attributes

```python
class Dog:
    # Class attribute — shared by ALL Dog instances
    species = "Canis lupus familiaris"
    count = 0   # tracks how many dogs exist

    def __init__(self, name, breed):
        # Instance attributes — unique per dog
        self.name = name
        self.breed = breed
        Dog.count += 1   # modify class attribute

    @classmethod
    def get_count(cls):
        """Class method — receives the class, not an instance."""
        return f"Total dogs: {cls.count}"

    @staticmethod
    def is_valid_name(name):
        """Static method — no access to instance or class."""
        return isinstance(name, str) and len(name) > 0

d1 = Dog("Rex", "German Shepherd")
d2 = Dog("Buddy", "Labrador")

print(d1.species)        # Canis lupus familiaris
print(Dog.species)       # Canis lupus familiaris
print(Dog.get_count())   # Total dogs: 2
print(Dog.is_valid_name("Rex"))  # True
```

---

## 🏛️ Inheritance — Building on Existing Classes

Inheritance lets one class **inherit** attributes and methods from another.

```python
class Animal:
    """Base class for all animals."""

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def speak(self):
        raise NotImplementedError("Subclass must implement speak()")

    def describe(self):
        return f"{self.name} is {self.age} years old"

    def __str__(self):
        return f"{self.__class__.__name__}({self.name})"


class Dog(Animal):
    """Dog inherits from Animal."""

    def __init__(self, name, age, breed):
        super().__init__(name, age)   # call parent's __init__
        self.breed = breed

    def speak(self):
        return f"{self.name} says: Woof!"

    def fetch(self, item):
        return f"{self.name} fetches the {item}!"


class Cat(Animal):
    def __init__(self, name, age, indoor=True):
        super().__init__(name, age)
        self.indoor = indoor

    def speak(self):
        return f"{self.name} says: Meow!"

    def purr(self):
        return f"{self.name} purrs..."


# Usage
rex = Dog("Rex", 3, "German Shepherd")
whiskers = Cat("Whiskers", 5)

print(rex.speak())           # Rex says: Woof!
print(whiskers.speak())      # Whiskers says: Meow!
print(rex.describe())        # Rex is 3 years old  (inherited method!)
print(rex.fetch("ball"))     # Rex fetches the ball!

# Polymorphism — treat different types the same
animals = [rex, whiskers, Dog("Buddy", 2, "Lab"), Cat("Luna", 4)]
for animal in animals:
    print(animal.speak())    # each calls its own version
```

---

## 🔍 Encapsulation — Protecting Data

Encapsulation means keeping an object's internal data private and controlled.

```python
class BankAccount:
    """A bank account with controlled balance access."""

    def __init__(self, owner, initial_balance=0):
        self.owner = owner
        self._balance = initial_balance   # _ means "private by convention"
        self.__transactions = []           # __ means "name-mangled" (truly private)

    @property
    def balance(self):
        """Read-only property — getter."""
        return self._balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        self.__transactions.append(f"+{amount}")
        return self._balance

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self._balance:
            raise ValueError("Insufficient funds")
        self._balance -= amount
        self.__transactions.append(f"-{amount}")
        return self._balance

    def get_history(self):
        return list(self.__transactions)

    def __str__(self):
        return f"Account({self.owner}: ₹{self._balance:,.2f})"


account = BankAccount("Alice", 1000)
account.deposit(500)
account.withdraw(200)

print(account.balance)         # 1300  (via property)
print(account)                 # Account(Alice: ₹1,300.00)
print(account.get_history())   # ['+500', '-200']

# account.balance = 9999  # AttributeError — can't set (read-only property)
```

---

## 🎭 Special (Dunder) Methods

Special methods let your objects work with Python's built-in operators.

```python
class Vector:
    """2D mathematical vector."""

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    def __repr__(self):
        return f"Vector(x={self.x}, y={self.y})"

    def __add__(self, other):
        """v1 + v2"""
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        """v1 - v2"""
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        """v * scalar"""
        return Vector(self.x * scalar, self.y * scalar)

    def __eq__(self, other):
        """v1 == v2"""
        return self.x == other.x and self.y == other.y

    def __len__(self):
        """len(v) — magnitude (integer)"""
        return int((self.x**2 + self.y**2) ** 0.5)

    def __abs__(self):
        """abs(v) — magnitude (float)"""
        return (self.x**2 + self.y**2) ** 0.5


v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(v1 + v2)    # Vector(4, 6)
print(v1 - v2)    # Vector(2, 2)
print(v1 * 3)     # Vector(9, 12)
print(v1 == v2)   # False
print(abs(v1))    # 5.0  (3-4-5 right triangle)
```

---

## 🧪 Data Science OOP Example — Custom Dataset Class

```python
class Dataset:
    """
    A simple dataset class that demonstrates OOP for Data Science.
    Wraps a list of records with useful methods.
    """

    def __init__(self, data=None, name="Dataset"):
        self.name = name
        self._data = data if data is not None else []

    def __len__(self):
        return len(self._data)

    def __getitem__(self, index):
        return self._data[index]

    def __iter__(self):
        return iter(self._data)

    def add(self, record):
        self._data.append(record)
        return self

    def filter(self, condition_func):
        """Return new Dataset with records matching condition."""
        filtered = [r for r in self._data if condition_func(r)]
        return Dataset(filtered, f"{self.name}_filtered")

    def map(self, transform_func):
        """Return new Dataset with transform applied to each record."""
        return Dataset([transform_func(r) for r in self._data], self.name)

    def column(self, key):
        """Extract a single column as a list."""
        return [r[key] for r in self._data if key in r]

    def summary(self):
        print(f"Dataset: {self.name}")
        print(f"Records: {len(self)}")
        if self._data:
            print(f"Keys: {list(self._data[0].keys())}")

    def __str__(self):
        return f"Dataset({self.name}, {len(self)} records)"


# Usage
records = [
    {"name": "Alice", "age": 30, "salary": 80000},
    {"name": "Bob", "age": 25, "salary": 55000},
    {"name": "Charlie", "age": 35, "salary": 120000},
    {"name": "Diana", "age": 28, "salary": 70000},
]

ds = Dataset(records, "employees")
ds.summary()

# Filter high earners
high_earners = ds.filter(lambda r: r["salary"] > 70000)
print(high_earners)   # Dataset(employees_filtered, 2 records)

# Extract ages
ages = ds.column("age")
print(f"Average age: {sum(ages)/len(ages):.1f}")
```

---

## ✅ Key Takeaways

- A **class** is a blueprint; an **object** is an instance of that blueprint
- `__init__` is the constructor — initializes instance attributes
- `self` refers to the current instance
- **Inheritance** (`class Child(Parent)`) lets you reuse and extend code
- `super().__init__()` calls the parent class constructor
- **Encapsulation** uses `_private` convention to protect internal state
- **Dunder methods** (`__str__`, `__add__`, etc.) make your objects feel native
- Scikit-learn models follow a consistent OOP pattern: `fit()`, `transform()`, `predict()`

---

## 🔗 What's Next?

➡️ [[02-file-handling]] — Read and write real-world data files