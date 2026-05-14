# 📄 08 – Python Basics Cheat Sheet

> Quick reference — bookmark this and keep it open while coding!

---

## Variables & Types

```python
x = 42              # int
y = 3.14            # float
s = "hello"         # str
b = True            # bool
n = None            # NoneType

type(x)             # <class 'int'>
isinstance(x, int)  # True
int("42")           # 42
float("3.14")       # 3.14
str(42)             # "42"
```

## Strings

```python
s = "Hello, World!"
s.upper()           # "HELLO, WORLD!"
s.lower()           # "hello, world!"
s.strip()           # remove whitespace
s.split(",")        # ['Hello', ' World!']
",".join(["a","b"]) # "a,b"
s.replace("o","0")  # "Hell0, W0rld!"
len(s)              # 13
s[0]                # 'H'
s[-1]               # '!'
s[0:5]              # 'Hello'
f"Val: {x:.2f}"     # f-string formatting
```

## Control Flow

```python
if x > 0:           # if
    pass
elif x == 0:        # else if
    pass
else:               # else
    pass

y = "pos" if x > 0 else "neg"   # ternary

for i in range(5):  # 0,1,2,3,4
    pass

for i, v in enumerate(lst):     # index + value
    pass

for a, b in zip(l1, l2):        # parallel iteration
    pass

while condition:    # while loop
    break           # exit loop
    continue        # skip to next
```

## Functions

```python
def func(req, opt=10, *args, **kwargs):
    return req

result = func("a", 20, 1, 2, x=5)

square = lambda x: x**2         # lambda
```

## Lists

```python
lst = [1, 2, 3]
lst.append(4)       # [1,2,3,4]
lst.extend([5,6])   # [1,2,3,4,5,6]
lst.insert(0, 0)    # [0,1,2,3,4,5,6]
lst.remove(3)       # removes first 3
lst.pop()           # removes+returns last
lst.sort()          # sort in place
sorted(lst)         # returns new sorted list
lst[::-1]           # reversed copy
len(lst)            # length
[x**2 for x in lst if x>2]     # comprehension
```

## Dictionaries

```python
d = {"a": 1, "b": 2}
d["c"] = 3          # add/update
d.get("z", 0)       # safe access, default 0
d.keys()            # keys
d.values()          # values
d.items()           # (key, value) pairs
del d["a"]          # delete key
d.pop("b")          # remove + return
{k: v*2 for k,v in d.items()}  # comprehension
```

## Tuples & Sets

```python
t = (1, 2, 3)       # immutable
a, b, c = t         # unpacking

s = {1, 2, 3, 2}    # {1, 2, 3} — unique
s1 | s2             # union
s1 & s2             # intersection
s1 - s2             # difference
x in s              # O(1) membership
```

## Built-in Functions

```python
len(x)      sum(x)      min(x)      max(x)
abs(x)      round(x,2)  sorted(x)   reversed(x)
zip(a,b)    enumerate(x)map(f,x)    filter(f,x)
any(x)      all(x)      range(n)    print(x)
type(x)     isinstance(x, T)        id(x)
```