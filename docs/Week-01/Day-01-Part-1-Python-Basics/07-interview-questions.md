# 🎯 07 – Interview Questions: Python Basics

> These are real questions asked in Data Science and Python developer interviews.

---

## 🔢 Variables & Data Types

**Q1:** What is the difference between `is` and `==` in Python?

> **Answer:** `==` compares **values**; `is` compares **identity** (same object in memory).
> ```python
> a = [1, 2, 3]
> b = [1, 2, 3]
> print(a == b)   # True  (same values)
> print(a is b)   # False (different objects)
> 
> c = a           # c points to the same object
> print(a is c)   # True
> ```
> **Rule:** Use `is` only for `None`, `True`, `False` comparisons. Use `==` for value comparisons.

---

**Q2:** What is the output of `0.1 + 0.2 == 0.3`? Why?

> **Answer:** `False`. Floating point numbers can't be represented exactly in binary. Use `abs(0.1 + 0.2 - 0.3) < 1e-9` to compare floats.

---

**Q3:** What is the difference between `int` and `float`? When does division return a float?

> **Answer:** `int` stores whole numbers; `float` stores decimals. In Python 3, `/` always returns a float (even `4 / 2 = 2.0`). Use `//` for integer division.

---

## 🔀 Control Flow

**Q4:** What is the difference between `break` and `continue`?

> **Answer:** `break` exits the loop entirely. `continue` skips the rest of the current iteration and jumps to the next one.

---

**Q5:** What does this code print?
> ```python
> for i in range(3):
>     pass
> print(i)
> ```
> **Answer:** `2` — the loop variable `i` retains its last value after the loop ends, even with `pass`.

---

**Q6:** What is a list comprehension? Write one that filters and transforms.

> **Answer:** A compact syntax for building lists. Example: get squares of even numbers from 1–20:
> ```python
> result = [x**2 for x in range(1, 21) if x % 2 == 0]
> ```

---

## 🔧 Functions

**Q7:** What is the difference between `*args` and `**kwargs`?

> **Answer:**
> - `*args` collects extra positional arguments into a **tuple**
> - `**kwargs` collects extra keyword arguments into a **dict**
> ```python
> def f(*args, **kwargs):
>     print(args)    # (1, 2, 3)
>     print(kwargs)  # {'x': 10, 'y': 20}
> f(1, 2, 3, x=10, y=20)
> ```

---

**Q8:** What is a lambda function? Write one that sorts a list of tuples by the second element.

> **Answer:** An anonymous one-line function.
> ```python
> data = [(1, 'banana'), (2, 'apple'), (3, 'cherry')]
> sorted_data = sorted(data, key=lambda x: x[1])
> # [(2, 'apple'), (1, 'banana'), (3, 'cherry')]
> ```

---

**Q9:** What is a mutable default argument trap?

> **Answer:** Using a mutable object (list, dict) as a default parameter is a classic bug:
> ```python
> # BUG — the list is created once and reused!
> def append_to(item, lst=[]):
>     lst.append(item)
>     return lst
> 
> print(append_to(1))   # [1]
> print(append_to(2))   # [1, 2]  ← unexpected!
> 
> # FIX — use None as default
> def append_to(item, lst=None):
>     if lst is None:
>         lst = []
>     lst.append(item)
>     return lst
> ```

---

## 📚 Data Structures

**Q10:** What is the difference between a list and a tuple?

> **Answer:**
> | | List | Tuple |
> |---|------|-------|
> | Mutable | ✅ | ❌ |
> | Syntax | `[1,2,3]` | `(1,2,3)` |
> | Use case | Dynamic data | Fixed data, dict keys |
> | Speed | Slightly slower | Slightly faster |

---

**Q11:** How do you safely get a value from a dict that might not have the key?

> **Answer:** Use `.get()` with a default:
> ```python
> d = {"name": "Alice"}
> email = d.get("email", "no email")   # "no email"
> ```

---

**Q12:** What is the time complexity of `in` for a list vs a set?

> **Answer:** List: O(n) — must check every element. Set: O(1) — uses a hash table. For large membership tests, always use a set.

---

**Q13:** How do you remove duplicates from a list while preserving order?

> **Answer:**
> ```python
> data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
> unique = list(dict.fromkeys(data))
> # [3, 1, 4, 5, 9, 2, 6]
> ```

---

**Q14:** What is the difference between `append()` and `extend()`?

> **Answer:**
> ```python
> a = [1, 2, 3]
> a.append([4, 5])     # [1, 2, 3, [4, 5]]  ← adds list as one element
> 
> b = [1, 2, 3]
> b.extend([4, 5])     # [1, 2, 3, 4, 5]    ← adds each element
> ```