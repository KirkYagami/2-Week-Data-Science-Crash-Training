# Matplotlib Basics — The Figure/Axes Object Model

Most data scientists learn Matplotlib the wrong way. They call `plt.plot()`, get something on screen, and never understand why their code breaks the moment they need two subplots or want to style one chart differently from another. This note covers the right mental model — the one that makes everything else click.

## Learning Objectives

- Explain the difference between the pyplot state-machine API and the object-oriented API
- Create figures and axes with `plt.subplots()` and control layout
- Set figure size and DPI for different output targets (screen vs. print vs. slides)
- Apply the OO API (`ax.plot()`, `ax.set_title()`, etc.) consistently
- Save publication-quality figures with `savefig()`

---

## The Mental Model You Actually Need

Matplotlib has two APIs. Most tutorials teach the wrong one first.

**The state-machine API** (`plt.plot()`, `plt.title()`, etc.) works by keeping track of a "current" figure and "current" axes behind the scenes. It feels convenient until you have multiple subplots. Then you lose track of which axes is "current" and the code breaks in non-obvious ways.

**The object-oriented API** (`fig, ax = plt.subplots()`, then `ax.plot()`) gives you explicit handles. You always know which figure and which axes you are modifying.

> [!info] The Two-Level Hierarchy
> Every Matplotlib chart has two objects you need to know:
> - **Figure** — the entire window or image file. Controls overall size, DPI, background.
> - **Axes** — a single plot area within the figure. Has its own x-axis, y-axis, title, labels, and data.
> One figure can contain many axes (subplots). One axes belongs to exactly one figure.

---

## Starting the Right Way: `plt.subplots()`

This is the line you should start every Matplotlib chart with:

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(8, 4))

x = np.linspace(0, 10, 100)
y = np.sin(x)

ax.plot(x, y)
ax.set_title("Sine Wave")
ax.set_xlabel("x")
ax.set_ylabel("sin(x)")

plt.tight_layout()
plt.savefig("sine_wave.png", dpi=150)  # or plt.show()
```

`fig` is the figure. `ax` is the axes. You call everything on `ax` from here on. That's it.

> [!warning] The Axes vs. Figure Confusion
> `plt.title()` sets the title on the *currently active* axes — fine for one plot, unpredictable for two.
> `ax.set_title()` sets the title on *that specific* axes object — always correct.
> When you have more than one subplot, always use the OO API. Make it a habit to always use it.

---

## Figure Size and DPI

`figsize` is in inches. `dpi` is dots per inch. The pixel dimensions of your output are `figsize * dpi`.

```python
import matplotlib.pyplot as plt
import numpy as np

# For a presentation slide (wide, lower resolution)
fig, ax = plt.subplots(figsize=(12, 5), dpi=100)

months = np.arange(1, 13)
revenue = [82, 91, 104, 118, 127, 143, 151, 148, 139, 162, 175, 198]

ax.plot(months, revenue, marker="o", linewidth=2)
ax.set_title("Monthly Revenue — 2024", fontsize=14)
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (thousands)")
ax.set_xticks(months)
ax.set_xticklabels(["Jan","Feb","Mar","Apr","May","Jun",
                     "Jul","Aug","Sep","Oct","Nov","Dec"])

plt.tight_layout()
plt.savefig("revenue_presentation.png", dpi=100)  # or plt.show()
```

> [!tip] DPI Guidelines
> - `dpi=72` — web/screen display
> - `dpi=150` — general purpose, Jupyter notebooks
> - `dpi=300` — print or publication quality
> - `dpi=100` with `figsize=(16, 9)` — presentation slides at 1600×900 px

---

## Multiple Subplots

`plt.subplots(rows, cols)` returns a figure and an array of axes.

```python
import matplotlib.pyplot as plt
import numpy as np

np.random.seed(42)
months = np.arange(1, 13)
revenue = [82, 91, 104, 118, 127, 143, 151, 148, 139, 162, 175, 198]
costs   = [60, 65,  72,  80,  88,  95, 102,  99,  94, 108, 115, 130]
profit  = [r - c for r, c in zip(revenue, costs)]

fig, axes = plt.subplots(1, 3, figsize=(14, 4))

# Each axes is addressed by index
axes[0].plot(months, revenue, color="#2563EB", marker="o", linewidth=2)
axes[0].set_title("Revenue")
axes[0].set_xlabel("Month")
axes[0].set_ylabel("Thousands")

axes[1].plot(months, costs, color="#DC2626", marker="s", linewidth=2)
axes[1].set_title("Costs")
axes[1].set_xlabel("Month")

axes[2].bar(months, profit, color="#16A34A")
axes[2].set_title("Profit")
axes[2].set_xlabel("Month")

plt.tight_layout()
plt.savefig("financial_summary.png", dpi=150)  # or plt.show()
```

> [!warning] Always Call `tight_layout()`
> Without `tight_layout()`, subplots frequently overlap — titles bleed into adjacent plots, axis labels get clipped. Always add it before `savefig()` or `show()`. If you are on a recent Matplotlib version, `fig.set_layout_engine("tight")` is the modern equivalent.

---

## 2×2 Grid of Subplots

For a 2D grid, `axes` becomes a 2D array. Access cells with `axes[row, col]`.

```python
import matplotlib.pyplot as plt
import numpy as np

np.random.seed(7)
x = np.random.normal(loc=50, scale=15, size=200)
y = 0.8 * x + np.random.normal(0, 8, 200)

fig, axes = plt.subplots(2, 2, figsize=(10, 8))

axes[0, 0].scatter(x, y, alpha=0.5, color="steelblue")
axes[0, 0].set_title("Scatter: x vs y")
axes[0, 0].set_xlabel("x")
axes[0, 0].set_ylabel("y")

axes[0, 1].hist(x, bins=20, color="coral", edgecolor="white")
axes[0, 1].set_title("Distribution of x")
axes[0, 1].set_xlabel("x")
axes[0, 1].set_ylabel("Count")

axes[1, 0].hist(y, bins=20, color="mediumseagreen", edgecolor="white")
axes[1, 0].set_title("Distribution of y")
axes[1, 0].set_xlabel("y")
axes[1, 0].set_ylabel("Count")

axes[1, 1].boxplot([x, y], labels=["x", "y"])
axes[1, 1].set_title("Boxplots: x and y")
axes[1, 1].set_ylabel("Value")

plt.suptitle("EDA Panel", fontsize=15, y=1.01)
plt.tight_layout()
plt.savefig("eda_panel.png", dpi=150)  # or plt.show()
```

---

## Styling: Colors, Markers, Linestyles

```python
import matplotlib.pyplot as plt
import numpy as np

t = np.linspace(0, 4 * np.pi, 80)

fig, ax = plt.subplots(figsize=(9, 4))

ax.plot(t, np.sin(t),   color="#0D9488", linestyle="-",  linewidth=2, marker="o",
        markersize=4, label="sin(t)")
ax.plot(t, np.cos(t),   color="#F59E0B", linestyle="--", linewidth=2, marker="s",
        markersize=4, label="cos(t)")
ax.plot(t, np.sin(t)/2, color="#6366F1", linestyle=":",  linewidth=1.5,
        label="sin(t)/2")

ax.set_title("Trigonometric Functions")
ax.set_xlabel("t (radians)")
ax.set_ylabel("Amplitude")
ax.legend(framealpha=0.9)
ax.grid(True, linestyle="--", alpha=0.5)

plt.tight_layout()
plt.savefig("trig_functions.png", dpi=150)  # or plt.show()
```

**Key styling parameters:**

| Parameter | Common Values |
|---|---|
| `color` | hex `"#0D9488"`, name `"steelblue"`, shorthand `"r"` |
| `linestyle` | `"-"` solid, `"--"` dashed, `":"` dotted, `"-."` dash-dot |
| `linewidth` | `1`, `1.5`, `2`, `2.5` |
| `marker` | `"o"` circle, `"s"` square, `"^"` triangle, `"x"` cross |
| `markersize` | `4`, `6`, `8` |
| `alpha` | `0.3`–`1.0` (transparency) |

---

## Saving Figures

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot([1, 2, 3, 4, 5], [10, 14, 12, 18, 22], marker="o", color="steelblue")
ax.set_title("Export Example")
ax.set_xlabel("Week")
ax.set_ylabel("Signups")

plt.tight_layout()

# PNG for web or reports
fig.savefig("chart.png", dpi=300, bbox_inches="tight")

# SVG for scalable graphics (presentations, Figma)
fig.savefig("chart.svg", bbox_inches="tight")
```

> [!tip] Use `fig.savefig()` not `plt.savefig()`
> When you have an explicit `fig` object, call `fig.savefig()`. It saves exactly the figure you built. `plt.savefig()` saves whatever matplotlib considers "current" — that can produce empty files if you've done anything to reset state since creating the figure.

> [!warning] `bbox_inches="tight"` Is Not Optional
> Without it, axis labels and titles routinely get clipped at the edges of the file. Always include it.

---

## The State-Machine API — When It's Acceptable

The pyplot API (`plt.plot()`, `plt.title()`) is fine for single, throwaway exploration in a notebook. You will see it everywhere, so you need to recognize it.

```python
import matplotlib.pyplot as plt

# Fine for one-off notebook exploration
plt.figure(figsize=(7, 3))
plt.plot([1, 2, 3, 4, 5], [3, 7, 5, 9, 11], color="teal", marker="o")
plt.title("Quick Look")
plt.xlabel("x")
plt.ylabel("y")
plt.tight_layout()
plt.savefig("quick_look.png")  # or plt.show()
```

The rule: use the OO API (`fig, ax = plt.subplots()`) whenever you have more than one subplot, are writing production code, or are building reusable chart functions.

---

## Practice Exercises

**Warm-up:** Create a single axes with `plt.subplots()`. Plot the sequence `[4, 7, 3, 9, 6, 11]` as a line chart with circular markers. Add a title, x-label, y-label, and save as `warmup.png`.

**Main:** Create a 1×2 subplot figure. Left: a line chart of `y = x**2` for x from −5 to 5. Right: a line chart of `y = x**3` for the same range. Give each its own title and axis labels. Use different colors. Call `tight_layout()` before saving.

**Stretch:** Create a 2×2 figure. In each subplot, plot `y = np.sin(k * x)` for k = 1, 2, 3, 4. Add a subtitle with `plt.suptitle()`. Make all lines different colors. Add a grid to each subplot. Save at 300 DPI.

---

## Interview Questions

**Q: What is the difference between a Figure and an Axes in Matplotlib?**

??? "Show answer"
    A Figure is the entire canvas — the outer container that holds everything. An Axes is a single plot area within the figure, with its own x-axis, y-axis, title, and data. One figure can hold multiple axes (subplots). Most Matplotlib properties that feel like "the chart" — labels, ticks, limits, plotted data — belong to the Axes, not the Figure.

**Q: Why is the OO API preferred over the pyplot state-machine API?**

??? "Show answer"
    The state-machine API tracks a "current figure" and "current axes" implicitly. With a single plot this works, but with multiple subplots you can accidentally modify the wrong axes. The OO API gives you explicit handles (`fig`, `ax`) so you always know exactly what you are modifying. It also makes charts easier to encapsulate in functions and easier to test.

**Q: What does `plt.tight_layout()` do and why is it important?**

??? "Show answer"
    `tight_layout()` automatically adjusts subplot padding so that titles, labels, and tick marks do not overlap with each other or get clipped. Without it, multi-panel figures frequently have elements bleeding into neighboring subplots. Always call it before `savefig()` or `show()`.

**Q: What does `dpi` control in `savefig()`?**

??? "Show answer"
    DPI (dots per inch) controls the pixel density of the saved image. `figsize` is in inches; the final image dimensions in pixels are `figsize * dpi`. Use 72–100 for screen/web, 150 for general notebooks, and 300 for print or publication.

---

> [!success] Key Takeaways
> - Always start with `fig, ax = plt.subplots()` — it gives you explicit control over every element.
> - The Figure is the canvas. The Axes is the plot. They are different objects.
> - Use `ax.set_title()`, `ax.set_xlabel()`, `ax.set_ylabel()` — not the `plt.*` equivalents — when you have multiple subplots.
> - Always call `tight_layout()` before saving.
> - `bbox_inches="tight"` in `savefig()` prevents label clipping.

---

[[02-line-bar-histogram|Next: Line, Bar, and Histogram Charts]]
