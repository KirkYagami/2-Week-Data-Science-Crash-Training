# Python Introduction

Python is the most widely used language in data science — not because it is the fastest, but because it lets you move from idea to working code faster than any alternative. Every hour you invest in understanding Python properly will pay back tenfold when you are building data pipelines and models.

---

## Learning Objectives

By the end of this note, you will be able to:

- Explain why Python became the dominant data science language
- Describe how the Python interpreter executes your code
- Run your first Python programs from a file or notebook
- Follow PEP 8 style conventions from day one
- Set up a working Python environment using any of three approaches

---

## What Python Is

Python is a high-level, interpreted, general-purpose programming language first released in 1991. "High-level" means it abstracts away memory management and other low-level machine concerns. You focus on solving the problem; Python handles the machinery underneath.

The name comes not from the snake but from *Monty Python's Flying Circus* — which explains why the community has a somewhat irreverent, practical culture.

To see why readability matters, compare the same task across languages:

```cpp
// C++ — printing "Hello, World!"
#include <iostream>
int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

```java
// Java — printing "Hello, World!"
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```python
# Python — printing "Hello, World!"
print("Hello, World!")
```

That single line does the same job. When you are exploring data and testing hypotheses all day, this economy of expression matters.

---

## Why Python Dominates Data Science

Python did not become the standard by accident. There are concrete reasons it beat MATLAB, R, Scala, and others for general-purpose data work.

### An ecosystem built specifically for data

| Library | What it does |
|---------|-------------|
| NumPy | Fast numerical arrays — the foundation for everything else |
| Pandas | Tabular data manipulation, like Excel but programmable |
| Matplotlib / Seaborn | Visualization and charts |
| Scikit-learn | Machine learning algorithms with a consistent API |
| TensorFlow / PyTorch | Deep learning and neural networks |
| SciPy | Scientific computing: optimization, statistics, signal processing |
| Statsmodels | Statistical modeling and hypothesis testing |

### Speed of development beats speed of execution

Python code is slower to run than C++. For 99% of data science tasks, this does not matter. Your bottleneck is rarely computation time on a single machine — it is the time to iterate on ideas. Python lets you iterate fast.

When raw speed does matter, NumPy and Pandas delegate to C code under the hood, so operations on large arrays run at compiled-language speed.

### Jupyter Notebooks changed the workflow

Data science is exploratory. You form a hypothesis, run some code, look at the output, refine your thinking, and run more code. Jupyter Notebooks — documents that interleave live code, output, and text — are perfectly shaped for this loop. No other language has a comparable, as-widely-adopted interactive environment.

### One language across the full pipeline

Data collection, cleaning, analysis, modeling, and deployment can all happen in Python. In many organizations the data scientists and the engineers who deploy the model speak the same language. That reduces friction.

> [!info]
> R is excellent for statistics and has better built-in statistical functions than Python. Many statisticians prefer it. Python wins for end-to-end production work because you can use the same language from data ingestion to model serving.

---

## How Python Runs Your Code

Understanding this helps you interpret error messages and performance characteristics.

```
Your Code (.py file or Jupyter cell)
              |
              v
    Python Interpreter (CPython)
              |
              v
      Bytecode (.pyc file, cached)
              |
              v
  Python Virtual Machine (PVM)
              |
              v
         Output / Result
```

Python compiles your source code to bytecode first, then executes that bytecode in the PVM. This compilation step happens automatically — you never invoke it manually. The `.pyc` files you sometimes see in a `__pycache__` folder are that cached bytecode.

### Interpreted vs compiled — what it means in practice

| | Python | C++ |
|--|--------|-----|
| Error detection | At runtime | At compile time |
| Development speed | Fast | Slow |
| Execution speed | Slower | Faster |
| Type errors | Surface when that line runs | Surface before the program runs |

The key implication: a bug in an untested code path in Python can stay hidden for months. This is why tests matter.

> [!warning]
> Python only runs the lines it reaches. A `TypeError` in a function no one calls will not surface until that function is called. Write small tests for your functions, even during exploration.

---

## Setting Up Your Environment

### Option 1: VS Code with Python extension (recommended for production work)

1. Install Python 3.10+ from [python.org](https://python.org)
2. Install VS Code from [code.visualstudio.com](https://code.visualstudio.com)
3. Install the Python extension from the VS Code extensions panel
4. Create `hello.py`, type `print("hello")`, and run it with the play button

### Option 2: Anaconda (recommended for data science)

Anaconda bundles Python, Jupyter, and all major data science libraries. Nothing to install separately.

1. Download from [anaconda.com](https://anaconda.com)
2. Launch Jupyter Notebook or JupyterLab from the Anaconda Navigator
3. Create a new notebook and start coding

### Option 3: Google Colab (zero setup)

The fastest way to start when you do not want to install anything.

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Create a new notebook
3. Every major data science library is pre-installed

> [!tip]
> Use Colab when you are learning or prototyping. Use VS Code or a local Jupyter setup for real projects where you need version control and reproducible environments.

---

## Your First Python Programs

### Print and arithmetic

```python
print("Hello, World!")
# Output: Hello, World!

# Arithmetic operators
print(2 + 3)     # Output: 5   — addition
print(10 - 4)    # Output: 6   — subtraction
print(3 * 7)     # Output: 21  — multiplication
print(15 / 4)    # Output: 3.75 — division (always float in Python 3)
print(15 // 4)   # Output: 3   — floor division (discard remainder)
print(15 % 4)    # Output: 3   — modulo (remainder)
print(2 ** 8)    # Output: 256 — exponentiation
```

### Storing and displaying values

```python
radius = 7
pi = 3.14159265
area = pi * radius ** 2

print(f"Radius: {radius}")
print(f"Area: {area:.2f}")
# Output:
# Radius: 7
# Area: 153.94
```

### Getting input from a user

```python
name = input("Enter your name: ")
print(f"Hello, {name}!")
```

> [!warning]
> `input()` always returns a string. If you ask for a number, you must convert it: `age = int(input("Enter age: "))`. Forgetting this is one of the most common beginner bugs.

---

## Comments — Talking to Humans, Not Computers

Comments are lines Python ignores. They exist to communicate intent to whoever reads the code later — usually you, six months from now.

```python
# Single-line comment — Python ignores everything after the #

# Calculate compound interest
principal = 10000
rate = 0.08        # annual interest rate
years = 5
final_amount = principal * (1 + rate) ** years
print(f"Final amount: {final_amount:.2f}")
# Output: Final amount: 14693.28
```

Write comments that explain *why*, not *what*. The code already shows what it does.

```python
# Bad comment — just restates the code
count = count + 1  # add 1 to count

# Good comment — explains the reason
count = count + 1  # compensate for the header row included in raw_row_count
```

Triple-quoted strings are sometimes used as multiline comments, though technically they are strings:

```python
"""
This function implements the Box-Cox transformation.
We use it here to reduce skew before fitting a linear model,
because linear regression assumes normally distributed residuals.
"""
```

---

## Python Style — PEP 8

PEP 8 is the official Python style guide. Following it means any Python developer can read your code immediately. Linters and formatters (like `flake8` and `black`) enforce these rules automatically.

| Rule | Avoid | Prefer |
|------|-------|--------|
| Variable names | `MyVar`, `myVar`, `MYVAR` | `my_var` |
| Spaces around operators | `x=1+2` | `x = 1 + 2` |
| Line length | Lines over 99 characters | Max 79–99 characters |
| Blank lines between functions | No separation | 2 blank lines |
| Imports | Mixed and tangled | One per line, at the top |

```python
# Poor style — hard to read
def calc_area(r):
    return 3.14159*r**2

# PEP 8 style — clean and readable
def calculate_circle_area(radius):
    pi = 3.14159
    return pi * radius ** 2
```

> [!tip]
> Install `black` (`pip install black`) and run `black yourfile.py` to auto-format your code to PEP 8 standards. Many teams run this automatically before every commit.

---

## The Python Ecosystem — Big Picture

```
             Your Data Science Code
                      |
       ┌──────────────┼──────────────┐
       |              |              |
    Pandas      Scikit-learn    Matplotlib
  (wrangling)    (modeling)    (visualization)
       |              |
    NumPy          NumPy
  (fast arrays)  (fast arrays)
       |
    Python
  (the foundation)
```

Everything in your data science work eventually calls down to Python fundamentals — variables, control flow, functions, and data structures. Mastering these is the highest-leverage thing you can do at the start of this course.

> [!success]
> Python is the foundation. NumPy and Pandas are built on it. Scikit-learn is built on NumPy. The further up the stack you go, the more you are hiding complexity — and the more you need to understand the layer below when something goes wrong.

---

## Key Takeaways

- Python is readable, expressive, and free — it became dominant in data science because of both its ecosystem and its development speed
- Python code is compiled to bytecode and run in the Python Virtual Machine — errors surface at runtime, not at compile time
- Use Jupyter Notebooks for exploration; `.py` files for reusable code and production scripts
- Follow PEP 8 from day one — it is a professional habit that makes your code readable to everyone

---

[[00-agenda]] | [[02-variables-and-data-types]]
