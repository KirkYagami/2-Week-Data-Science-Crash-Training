# Day 02 — Part 1: NumPy Fundamentals
## Agenda & Learning Roadmap

> [!info] What You Are Walking Into
> NumPy is not just a library. It is the memory model, the speed contract, and the coordinate system that every serious numerical tool in Python is built on. Pandas DataFrames sit on NumPy arrays. Scikit-learn models expect NumPy arrays. TensorFlow and PyTorch mimic NumPy's API. The hour you spend truly understanding NumPy pays compound interest for your entire data science career.

---

## Today's Roadmap

```
NumPy Fundamentals
│
├── 01 → Introduction to NumPy
│         Why Python lists fail at scale | The ndarray concept
│         Memory layout | dtype system and why it matters
│
├── 02 → Array Creation
│         np.array | zeros/ones/full | arange vs linspace (when each)
│         Random module | eye/diag | zeros_like | reshape
│
├── 03 → Indexing & Slicing
│         1D / 2D / 3D indexing | Negative indices | Step slicing
│         Boolean indexing (the workhorse) | Fancy indexing | np.where
│         Views vs Copies — the silent bug factory
│
├── 04 → Vectorization
│         For-loop vs vectorized timing | Universal functions (ufuncs)
│         Axis-wise aggregation | Why it is fast (SIMD, cache locality)
│
├── 05 → Broadcasting
│         The 3 rules | Visual intuition | Row and column broadcasting
│         Practical patterns | Shape errors and how to read them
│
├── 06 → Exercises
│         Warm-up → Main → Stretch — realistic data science problems
│
└── 07 → Cheat Sheet
          Dense quick-reference with code for every operation
```

---

## Time Allocation

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

## Learning Objectives

By the end of this session, you will be able to:

- Explain why NumPy arrays are faster than Python lists and what makes that possible at the hardware level
- Create arrays using the right function for the situation — not just the one you remember first
- Access and modify array data using indexing, slicing, boolean masks, and fancy indexing
- Predict whether an operation will return a view or a copy, and explain why this matters
- Replace explicit for-loops with vectorized operations and measure the speedup
- Apply broadcasting to operate on differently-shaped arrays without writing extra code
- Read a broadcasting error message and fix the shape mismatch

---

## Prerequisites

- Python basics: variables, lists, loops, functions
- Python installed with NumPy (`pip install numpy` or `conda install numpy`)
- A way to run code: Jupyter Notebook, VS Code, or a Python script

---

## Setup Check

Run this before the session:

```python
import numpy as np
print(np.__version__)  # 1.24 or higher is fine
```

> [!tip] Version Note
> NumPy 1.24+ and 2.x are both fine for everything in these notes. The `np.random.default_rng()` API used here requires 1.17+.

---

## Files In This Module

| File | Topic |
|------|-------|
| `01-introduction-to-numpy.md` | Why NumPy, the ndarray, memory layout, dtype |
| `02-array-creation.md` | Every way to build arrays, when to use each |
| `03-indexing-and-slicing.md` | Accessing data, views vs copies, boolean indexing |
| `04-vectorization.md` | Fast math without loops, ufuncs, axis operations |
| `05-broadcasting.md` | Operating on different-shaped arrays with visual intuition |
| `06-numpy-exercises.md` | Practice problems — warm-up, main, stretch |
| `07-cheat-sheet.md` | Dense quick-reference |

---

## How to Use These Notes

> [!tip] Study Strategy
> Type every code example yourself — do not paste. The mistakes you make while typing teach you more than reading does. After each section, close the notes and try to reproduce the key ideas from memory. Do the exercises before checking solutions. Struggling for ten minutes before looking at the answer is worth more than reading it cold.

---

[[01-introduction-to-numpy]]
