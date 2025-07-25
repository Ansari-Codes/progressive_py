# progressive_py Progress Bar Module

This module provides customizable progress bars for terminal and notebook environments. It supports single, nested, and parallel progress bars with rich styling and flexible configuration.

---

## Classes

### PBAR

The core progress bar class.

**Initialization Arguments (`args` dictionary):**
- `bar`: Character(s) for the filled part of the bar (default `'█'`)
- `space`: Character(s) for the empty part (default `' '`)
- `colors`: List of colors for the bar (default `['white']`)
- `color`: Single color for the bar (used if `colors` not provided)
- `paint`: Painting style (`'bar'` or `'bar-by-bar'`)
- `line`: Vertical position for stacking multiple bars
- `txt_lf` / `text_lf`: Left-side text template
- `txt_rt` / `text_rt`: Right-side text template
- `fg`: Foreground text colors, as list [left_fg, right_fg]
- `bg`: Background text colors, as list [left_bg, right_bg]
- `length`: Length of the bar (default `20`)
- `head`: Character(s) for the bar "head"
- `head_clr`: Color for the head
- `space_clr`: Color for the empty space
- `freq`: Minimum update interval (seconds, default `0.1`)
- `tokens`: Custom template tokens for text

**Key Methods:**
- `update(progress, current_iter, total_iter)`: Updates the bar display.
- `hide()`: Hides the bar.
- `show()`: Shows the bar.
- Context manager support (`__enter__`, `__exit__`).

---

### ProgressBar

A subclass of `PBAR` with throttled updates for performance. It is preferred.

---

## Functions

### nested_bar(main_bar, task_funcs, delay=0.05, auto_arrangement=True)

Runs multiple subtasks with individual progress bars and an overall parent bar.

**Arguments:**
- `main_bar`: ProgressBar instance for overall progress.
- `task_funcs`: Dictionary `{task_name: [function, bar_args_dict]}`
- `delay`: Delay between task transitions (seconds).
- `auto_arrangement`: If `True`, bars are stacked automatically; otherwise, specify `line` in each bar’s args.

**Usage Example:**
```python
def task1(bar): # Each function should update the bar internally
    for i in range(100):
        time.sleep(0.01)
        bar.update(i/99, i+1, 100) # in this way

def task2(bar):
    for i in range(50):
        time.sleep(0.01)
        bar.update(i/49, i+1, 50)
# child dict is passed in way: str:list[function, child_bar_args]
main = ProgressBar({'line':2})
childs = {
    "Task1": [task1, {"txt_lf": "Task1 {percent:.0f}%"}],
    "Task2": [task2, {"txt_lf": "Task2 {percent:.0f}%"}]
}
nested_bar(main, childs)
```

---

### simple(iterable, args={}, **kwargs)

Wraps an iterable with a progress bar.

**Arguments:**
- `iterable`: Iterable to wrap.
- `args`: Progress bar configuration dictionary.
- `kwargs`: Additional configuration overrides.

**Usage Example:**
```python
from progressive_py.progress_bar import simple

for i in simple(range(50), txt_lf="Step {iters} [{percent:.0f}%]"):
    time.sleep(0.05)
```

---

## Text Templates

You can use tokens in `txt_lf` and `txt_rt`:
- `{percent}`: Progress percent
- `{iters}`: Current/total iterations
- `{eta}`: Estimated time remaining
- `{elapsed}`: Elapsed time
- `{speed}`: Iterations per second

Custom tokens can be added via the `tokens` argument. Here is an example:
```python
from progressive_py.progress_bar import simple, time, ProgressBar

def inject_custom(bar:ProgressBar):
    return files[int(bar.current_iter)-1]

files = ['pack1', 'pack2', 'pack3', 'pack4']
for i in simple(range(4),
        txt_lf="Installing {pack} [{percent:.0f}]", 
        tokens = {"pack":inject_custom}):
    time.sleep(0.5)
```

There should be a function which returns the value of token, and that function should always accept the bar argument. Then, in tokens paramter in simple generator, pass as `{token_name:token_function}`.

---

## Notes

- For nested/parallel bars, use the `line` argument to control vertical stacking.
- Colors and styles are handled via `colorama`.