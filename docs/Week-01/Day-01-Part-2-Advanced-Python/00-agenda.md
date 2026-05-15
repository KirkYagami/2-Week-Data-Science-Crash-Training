# Day 01 – Part 2: Advanced Python

The difference between someone who can write Python and someone who can write *production Python* comes down to five things: how they model problems with objects, how they handle data files, how they structure code into reusable pieces, how they deal with failure gracefully, and whether their code is readable six months later. This session covers all five.

## Learning Objectives

By the end of this session, you will be able to:

- Build classes with constructors, inheritance, properties, and dunder methods
- Read and write CSV and JSON files safely without leaving file handles open
- Structure code into modules and understand Python's import system
- Handle errors specifically and deliberately — not with a blank `except:`
- Write code that reads like idiomatic Python, not a translation from Java

---

## Session Map

| # | Topic | File | Time |
|---|-------|------|------|
| 1 | OOP Basics | `01-oop-basics.md` | 35 min |
| 2 | File Handling | `02-file-handling.md` | 25 min |
| 3 | Modules & Packages | `03-modules-and-packages.md` | 20 min |
| 4 | Exception Handling | `04-exception-handling.md` | 25 min |
| 5 | Python Best Practices | `05-python-best-practices.md` | 20 min |
| 6 | Mini Exercises | `06-mini-exercises.md` | 30 min |
| 7 | Interview Questions | `07-interview-questions.md` | Reference |

**Total active learning time:** ~155 minutes (exercises included)

---

## Prerequisites

Complete [[../Day-01-Part-1-Python-Basics/04-functions|Functions]] and [[../Day-01-Part-1-Python-Basics/05-lists-tuples-dictionaries|Data Structures]] from Part 1 before starting here. You will be building on both.

---

## How to Use This Section

Read each file in order. Every concept has a runnable code example — type it out, don't copy-paste. The exercise file (`06-mini-exercises.md`) has three difficulty levels. Work through at least the warm-up and main problems before moving to Day 02.

The interview questions file (`07-interview-questions.md`) uses collapsible answers. Try answering each question yourself before revealing the model answer.

---

## Why This Matters for Data Science

Every serious data science library is built on these foundations:

- `scikit-learn` models are classes with `fit()`, `transform()`, and `predict()` methods
- Data pipelines read files, handle encoding errors, and skip corrupt rows
- Reproducible projects use module structure and `requirements.txt`
- Production code handles bad API responses, missing files, and malformed data — it does not crash

> [!success]
> Mastering these five topics puts you in a different category from analysts who only know Pandas. You become someone who can write tools, not just use them.

[[../Day-01-Part-1-Python-Basics/08-cheat-sheet|← Day 01 Part 1 Cheat Sheet]] | [[01-oop-basics|OOP Basics →]]
