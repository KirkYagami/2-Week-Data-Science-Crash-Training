# 🐍 01 – Python Introduction

> **Prerequisites:** None — this is the very beginning!  
> **Time to read:** ~20 minutes

---

## 🌟 What is Python?

Python is a **high-level, interpreted, general-purpose programming language** created by **Guido van Rossum** and first released in **1991**. The name comes not from the snake, but from the British comedy group *Monty Python's Flying Circus* — which explains Python's playful culture.

### Plain-English Definition

> Python is a language that lets you tell a computer what to do using instructions that look almost like English sentences.

Compare a simple task in different languages:

**C++ (complex, low-level):**
```cpp
#include <iostream>
int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

**Java (verbose):**
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Python (clean and simple):**
```python
print("Hello, World!")
```

That single line does the same job. This is the Python philosophy in action.

---

## 📊 Why Python for Data Science?

Python didn't become the #1 Data Science language by accident. Here are the concrete reasons:

### 1. Massive Ecosystem of Libraries

| Library | What it does |
|---------|-------------|
| **NumPy** | Fast numerical arrays & math |
| **Pandas** | Data manipulation (like Excel, but 1000× faster) |
| **Matplotlib / Seaborn** | Data visualization & charts |
| **Scikit-learn** | Machine learning algorithms |
| **TensorFlow / PyTorch** | Deep learning / neural networks |
| **SciPy** | Scientific computing |
| **Statsmodels** | Statistical modeling |

### 2. Readable Syntax

Python code reads like pseudocode. You spend less time fighting the language and more time solving problems.

### 3. Interactive Notebooks (Jupyter)

Data Scientists use **Jupyter Notebooks** — an environment where you can mix code, visualizations, and text in one document. This is perfect for exploration and storytelling with data.

### 4. Huge Community

- Over **8 million** Python developers worldwide
- Millions of answered questions on Stack Overflow
- Thousands of tutorials, courses, and books

### 5. Free & Open Source

Python and virtually all its Data Science libraries cost **$0**.

---

## 🔄 How Python Works — The Interpreter

Understanding *how* Python runs your code demystifies many errors.

```
Your Code (.py file)
        ↓
  Python Interpreter
        ↓
  Bytecode (.pyc)
        ↓
  Python Virtual Machine (PVM)
        ↓
  Output / Result
```

### Interpreted vs Compiled

| Feature | Python (Interpreted) | C++ (Compiled) |
|---------|---------------------|----------------|
| Speed | Slower to run | Faster to run |
| Development | Faster to write | Slower to write |
| Error detection | At runtime | At compile time |
| Portability | Very portable | Less portable |

> **Key takeaway:** Python is slower than C++, but for 99% of Data Science tasks, this doesn't matter — your bottleneck is your brain, not Python's speed. And NumPy/Pandas are backed by C code anyway, so they *are* fast.

---

## 🛠️ Setting Up Your Environment

### Option 1: Python + VS Code (Recommended for Beginners)

1. Download Python from [python.org](https://python.org)
2. Download VS Code from [code.visualstudio.com](https://code.visualstudio.com)
3. Install the **Python extension** in VS Code
4. Create a file `hello.py` and run it

### Option 2: Anaconda (Recommended for Data Science)

Anaconda bundles Python + all major Data Science libraries:
1. Download from [anaconda.com](https://anaconda.com)
2. Launch **Jupyter Notebook** or **JupyterLab**
3. Start coding immediately

### Option 3: Google Colab (Zero setup — works in browser)

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Create a new notebook
3. Start coding — no installation needed

---

## 📝 Your First Python Programs

### Hello World
```python
print("Hello, World!")
```
**Output:**
```
Hello, World!
```

### A simple calculation
```python
print(2 + 3)         # Addition → 5
print(10 - 4)        # Subtraction → 6
print(3 * 7)         # Multiplication → 21
print(15 / 4)        # Division → 3.75
print(15 // 4)       # Integer division → 3
print(15 % 4)        # Remainder (modulo) → 3
print(2 ** 8)        # Exponentiation → 256
```

### Getting user input
```python
name = input("What is your name? ")
print("Hello,", name)
```

---

## 💬 Comments — Talking to Humans, Not Computers

Comments are lines Python **ignores**. They exist to explain code to humans (including future-you).

```python
# This is a single-line comment

# Calculate the area of a circle
radius = 5
area = 3.14159 * radius ** 2  # pi * r^2
print(area)

"""
This is a multi-line comment (technically a string, but used as a comment).
Use it to explain long blocks of code or write documentation.
"""
```

> **Best practice:** Write comments that explain *why*, not *what*. The code already shows what it does.

```python
# BAD comment — just repeats the code
x = x + 1  # add 1 to x

# GOOD comment — explains the reason
x = x + 1  # compensate for 0-based indexing
```

---

## 🔠 Python Style Guide (PEP 8) — The Basics

PEP 8 is Python's official style guide. Following it makes your code readable to everyone.

| Rule | Bad | Good |
|------|-----|------|
| Variable names | `MyVar`, `MYVAR` | `my_var` |
| Spaces around operators | `x=1+2` | `x = 1 + 2` |
| Line length | 200 characters | Max 79–99 characters |
| Blank lines between functions | None | 2 blank lines |

```python
# BAD style
def calculateArea(r):
    return 3.14159*r**2

# GOOD style (PEP 8)
def calculate_area(radius):
    return 3.14159 * radius ** 2
```

---

## 🌐 The Python Ecosystem — Big Picture

```
        ┌──────────────────────────────────────┐
        │          Your Data Science Code      │
        └──────────────┬───────────────────────┘
                       │
     ┌─────────────────┼──────────────────┐
     │                 │                  │
  Pandas           Scikit-learn       Matplotlib
(Data wrangling)  (ML algorithms)   (Visualization)
     │                 │
   NumPy            NumPy
(Fast arrays)    (Fast arrays)
     │
  Python
(Foundation)
```

Everything in Data Science is built on Python. Master Python basics → master everything above it.

---

## ✅ Key Takeaways

- Python is **readable, powerful, and free** — perfect for Data Science
- Python code runs through an **interpreter** line by line
- The Python ecosystem (NumPy, Pandas, sklearn, etc.) is what makes it powerful for data work
- Use **Jupyter Notebooks** for data exploration; **.py files** for production code
- Follow **PEP 8** style from day one — it's a professional habit

---

## 🔗 What's Next?

➡️ [[02-variables-and-data-types]] — Learn how Python stores and organizes data