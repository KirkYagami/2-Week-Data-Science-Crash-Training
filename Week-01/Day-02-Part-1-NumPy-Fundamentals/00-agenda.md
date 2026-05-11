# 📅 Day 02 — Part 1: NumPy Fundamentals
## Agenda & Learning Roadmap

> [!info] Overview
> Welcome to **Day 2, Part 1** of your Data Science journey! Today we dive deep into **NumPy** — the foundational library that powers almost all scientific computing in Python. By the end of today, you'll think in arrays, not loops.

---

## 🗺️ Today's Roadmap

```
NumPy Fundamentals
│
├── 01 → Introduction to NumPy
│         Why NumPy? | Installing | Importing | ndarray basics
│
├── 02 → Array Creation
│         np.array | np.zeros | np.ones | np.arange | np.linspace
│         np.random | np.eye | np.full
│
├── 03 → Indexing & Slicing
│         1D | 2D | 3D | Boolean indexing | Fancy indexing
│
├── 04 → Vectorization
│         Element-wise ops | Universal Functions (ufuncs) | Performance
│
├── 05 → Broadcasting
│         Rules | Shape alignment | Real-world use cases
│
├── 06 → Exercises
│         Hands-on practice problems with solutions
│
└── 07 → Cheat Sheet
          Quick-reference for everything covered today
```

---

## ⏱️ Time Allocation

| # | Topic | Duration | Type |
|---|-------|----------|------|
| 01 | Introduction to NumPy | 20 min | Lecture |
| 02 | Array Creation | 25 min | Lecture + Code |
| 03 | Indexing & Slicing | 30 min | Lecture + Code |
| 04 | Vectorization | 20 min | Lecture + Demo |
| 05 | Broadcasting | 25 min | Lecture + Visuals |
| 06 | Exercises | 40 min | Hands-on Practice |
| 07 | Cheat Sheet Review | 10 min | Review |

**Total: ~2.5 hours**

---

## 🎯 Learning Objectives

By the end of this session, you will be able to:

- [ ] Explain what NumPy is and why it's faster than plain Python
- [ ] Create arrays using 8+ different NumPy functions
- [ ] Access and modify array elements using indexing and slicing
- [ ] Apply vectorized operations instead of writing for-loops
- [ ] Use broadcasting to operate on arrays of different shapes
- [ ] Write clean, efficient NumPy code confidently

---

## 🔧 Prerequisites

- [ ] Python basics (variables, lists, loops, functions)
- [ ] Anaconda / pip installed
- [ ] Jupyter Notebook or VS Code with Python extension

---

## 📦 Setup Check

Run this in your terminal to verify NumPy is installed:

```python
import numpy as np
print(np.__version__)  # Should print 1.24+ or higher
```

> [!tip] First Time?
> If NumPy isn't installed: `pip install numpy` or `conda install numpy`

---

## 📚 Files In This Module

| File | Topic |
|------|-------|
| `01-introduction-to-numpy.md` | What is NumPy, ndarray, memory model |
| `02-array-creation.md` | All the ways to create arrays |
| `03-indexing-and-slicing.md` | Accessing & modifying array data |
| `04-vectorization.md` | Fast math without loops |
| `05-broadcasting.md` | Operating on different-shaped arrays |
| `06-numpy-exercises.md` | Practice problems with solutions |
| `07-cheat-sheet.md` | Quick reference card |

---

## 💡 How to Use These Notes

> [!example] Study Strategy
> 1. Read the lecture notes **actively** — type every code example yourself
> 2. Don't copy-paste! Muscle memory matters in coding
> 3. After each section, close the notes and try to recall the key ideas
> 4. Do the exercises **before** looking at solutions
> 5. Keep the cheat sheet open during practice

---

*Let's build your NumPy foundation! 🚀*