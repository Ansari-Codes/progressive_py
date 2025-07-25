# progressive_py Notebook Progress Bar Module

This module provides rich, fully customizable progress bars for Jupyter and IPython notebook environments. It leverages HTML and CSS for advanced styling, supports dynamic updates, and is designed for both simple and complex workflows, including nested progress bars. (this is my favourite one!)

---

## Classes

### Bar

Base class for notebook progress bars.

**Initialization Arguments (`args` dictionary):**
- `id`: Unique identifier for the bar (auto-generated if not provided)
- `txt_lf`: Left-side text template (default `'Progress'`)
- `length`: Number of segments in the bar (default `40`)
- `bar`: Characters for filled segments (default `['█']`)
- `space`: Character for empty segments (default `' '`)
- `txt_rt`: Right-side text template (default `''`)
- `paint`: Painting style (`'bar-by-bar'`, `'bar'`, `'progress-bar'`)
- `colors`: List of CSS color values for bar segments (default `['green']`)
- `bar_style`: Inline CSS for bar segments
- `space_style`: CSS for empty segments (default `'color: #eee;'`)
- `progressbar_style`: CSS for the wrapper div (default includes monospace font, rounded corners, etc.)
- `add_css`: Additional CSS rules
- `text_color`: CSS color(s) for label text (default `'#eee'`)
- `tokens`: Custom format hooks for text templates
- `freq`: Minimum update interval in seconds (default `0.1`)

**Attributes:**
- `current_iter`: Current iteration count
- `total_iter`: Total iterations
- `progress`: Progress value (0 to 1)
- `bar_html`: Full HTML for the bar
- `visible`: Bar visibility
- `bar_segments`: Inline HTML for the bar only (no wrapper)

**Key Methods:**
- `update(progress, current_iter=None, total_iter=None)`: Updates the bar display in the notebook.
- `render(progress)`: Generates HTML for the current progress.
- `process_text_templates(text, progress)`: Replaces tokens in text templates with live stats (percent, ETA, speed, etc.).
- `calculate_eta_seconds()`, `calculate_speed_value()`, `format_time(seconds)`: Utility methods for progress stats.
- `show()`, `hide()`: Control bar visibility.
- `get_inline_html()`: Returns only the bar’s HTML (for embedding).
- Context manager support (`__enter__`, `__exit__`).

---

### NotebookProgressBar

Subclass of `Bar` optimized for Jupyter notebooks.

**Features:**
- Throttled updates for performance (`freq` argument).
- Full HTML/CSS rendering.
- Inherits all customization from `Bar`.

**Usage Example:**
```python
from progressive_py.ntbk_progbar import NotebookProgressBar
from progressive_py.utils import gradient_colors
from progressive_py.manager import AssetsManager
import time
style = AssetsManager().load('progress_bar', 'style', 'its_cool')
pb = NotebookProgressBar({
    **style,
    'txt_lf': "I'm Fast... [{percent:.0f}%] | ",
    'txt_rt': "| ETA {eta} | Elapsed {elapsed}",
    'colors': gradient_colors('#00ff00', '#0000ff', 15),
    'paint': 'bar-by-bar',
    'text_color': ['#efa', '#afe'],
    'length': 15,
})

total = 100
for i in range(total + 1):
    pb.update(i/ total, i, total)
    time.sleep(0.03)
print("Progress Complete!")
```

---

## Functions

### simple(iterable, total=None, args=None, **kwargs)

Generator for wrapping an iterable with a notebook progress bar.

**Arguments:**
- `iterable`: Any iterable object.
- `total`: Total number of iterations (required if iterable has no `__len__`).
- `args`: Progress bar configuration dictionary.
- `kwargs`: Additional configuration overrides.

**Usage Example:**
```python
for i in simple(range(100), txt_lf="Processing {iters} "):
    time.sleep(0.1)
```

---

### nested_bar(main_bar, task_funcs: dict, delay=0.05)

Runs multiple subtasks with individual notebook progress bars and an overall parent bar.

**Arguments:**
- `main_bar`: NotebookProgressBar instance for overall progress.
- `task_funcs`: Dictionary `{task_name: [function, bar_args_dict]}`
- `delay`: Delay between task transitions (seconds).

**Usage Example:**
```python
def task1(bar):
    for i in range(100):
        time.sleep(0.01)
        bar.update(i/99, i+1, 100)

def task2(bar):
    for i in range(50):
        time.sleep(0.01)
        bar.update(i/49, i+1, 50)

main = NotebookProgressBar()
childs = {
    "Task1": [task1, {"txt_lf": "Task1 {percent:.0f}%"}],
    "Task2": [task2, {"txt_lf": "Task2 {percent:.0f}%"}]
}
nested_bar(main, childs)
```

---

## Text Templates & Customization

- Use tokens in `txt_lf` and `txt_rt`:
  - `{percent}`: Progress percent
  - `{iters}`: Current/total iterations
  - `{eta}`: Estimated time remaining
  - `{elapsed}`: Elapsed time
  - `{speed}`: Iterations per second
- Add custom tokens via the `tokens` argument (dictionary of `{name: function}`).
- All styling is done via CSS, allowing gradients, animations, and more.

---

## Advanced Styling

- `colors`: List of CSS color values for each bar segment (supports gradients).
- `bar_style`, `space_style`, `progressbar_style`: Inline CSS for fine control.
- `add_css`: Inject custom CSS (e.g., keyframes, animations).
- `text_color`: List for left/right label colors.

---

## Notes

- Designed for Jupyter/IPython notebook environments.
- Uses `IPython.display` for live updates.
- Supports context manager usage for automatic cleanup.
- Handles both simple and nested progress bars.
- Fully customizable via HTML/CSS.
