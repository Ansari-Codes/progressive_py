# ProgressivePy


Elegant, customizable progress bars and spinners for Python — in both terminal and Jupyter notebook environments!

---

## Features

- **Highly customizable:** Colors, styles, templates, and more.
- **Terminal & Notebook support:** Works seamlessly in scripts, CLI, and Jupyter/IPython.
- **Nested & parallel bars:** Track multiple tasks at once.
- **Spinners:** Animated indicators for indeterminate tasks.
- **Low dependencies:** Lightweight and easy to install.
- **Thread management:** Run multiple bars/spinners concurrently.

---
## Installation from Git

```bash
pip install git+https://github.com/Ansari-Codes/progressive_py.git
```

---

## Quick Start

### Terminal Progress Bar

```python
from progressive_py.progress_bar import simple, time

for i in simple(range(100), txt_lf="Processing {iters} "):
    time.sleep(0.1)
```

### Styled Progress Bar

```python
from progressive_py.manager import AssetsManager
from progressive_py.progress_bar import simple, time
mgr = AssetsManager()
theme = mgr.load('progress_bar', 'theme', 'neon_wave')
style = mgr.load('progress_bar', 'style', 'its_cool')
for i in simple(range(100),
                dict(     # pass theme seperatly as dict, its safer
                    **theme,
                    **style
                ),
                head = '>', # change parameters of passed dict before
                txt_lf='You\'re being hacked... {percent:.0f}% |',
                txt_rt='| ETA: {eta} | Elapsed: {elapsed}'
                ):
    time.sleep(0.05)
```

### Terminal Spinner

```python
from progressive_py import spinner
import time
from progressive_py.manager import AssetsManager

mgr = AssetsManager()
style = mgr.load('spinner', 'style', 'dots')
theme = mgr.load('spinner', 'theme', 'sunset')
spinner = spinner.Spinner(seq=style, spn_side='left', text=' Loadings ', **theme)

spinner.start()
time.sleep(2)
spinner.stop()
```

### Notebook Progress Bar

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

### Notebook Spinner

```python
from progressive_py.ntbk_spinner import NotebookDivSpinner
import time

spn = NotebookDivSpinner({"text": "Loading"})
spn.start()
time.sleep(2)
spn.stop()
```

### Style Spinner

```python

from progressive_py.ntbk_spinner import NotebookDivSpinner
import time

spinner = NotebookDivSpinner(
    text="Loading...",
    final_text="Done!",
    speed=1.2,
    refresh=0.1,
    fg_text=["#2afadf", "#00c9ff", "#ff0080", "#7928ca"],
    bg_text=["#111111", "#222222", "#333333"],
    clr_interval=[0.4],
    container_css={
        "box-shadow": "0 4px 12px rgba(0, 0, 0, 0.25)",
        "background": "#1a1a1a",
        "border-radius": "10px",
        "padding": "6px 14px",
        "gap": "10px",
        "display": "inline-flex",
        "align-items": "center",
        "margin": "0px",  # No top margin
        "max-width": "fit-content"
    },
    spinner={
        "spinner_css": {
            "width": "20px",
            "height": "20px",
            "border": "4px solid #333",         # Full border
            "border-top": "4px solid #0ff",     # Spinner highlight
            "border-radius": "50%",
            "animation": "spin 1s linear infinite"
        },
        "text_css": {
            "font-size": "15px",
            "font-weight": "bold"
        }
    },
    animation={  # Keyframes!
        "spin 1s linear infinite":
        "0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); }"
    }
)
spinner.start()
for i in range(21):
    spinner.set_text(f"<b>Loading...</b> {i*5}%")
    time.sleep(0.5)
spinner.stop()
```

---

## Advanced Usage

### Nested Progress Bars

```python
from progressive_py.progress_bar import nested_bar, ProgressBar
import time

def task1(bar):
    for i in range(100):
        time.sleep(0.01)
        bar.update(i/99, i+1, 100)

def task2(bar):
    for i in range(50):
        time.sleep(0.01)
        bar.update(i/49, i+1, 50)

main = ProgressBar({'line':2})
childs = {
    "Task1": [task1, {"txt_lf": "Task1 {percent:.0f}%"}],
    "Task2": [task2, {"txt_lf": "Task2 {percent:.0f}%"}]
}
nested_bar(main, childs)
```

### Threaded Bars

```python
from progressive_py.progress_bar import ProgressBar
from progressive_py.utils import BarThreadManager
import time

def example_task(bar, pause_event, stop_check, cool_down=0.05):
    total = 100
    for i in range(total):
        if stop_check():
            break
        pause_event.wait()
        time.sleep(cool_down)
        bar.update(i / (total - 1), i + 1, total)

bar1 = ProgressBar({'txt_lf': 'Task 1 {percent:.0f}% {eta} {elapsed}', 'line': 1})
bar2 = ProgressBar({'txt_lf': 'Task 2 {percent:.0f}% {eta} {elapsed}', 'line': 0})

tasks = {
    "task1": [bar1, example_task],
    "task2": [bar2, example_task],
}

manager = BarThreadManager(tasks)
manager.start_all(cool_down=0.02)
manager.wait_all()
print("\nAll tasks completed.")
```

# See docs/* documentation for more info

---

## Customization

- **Text templates:** Use `{percent}`, `{iters}`, `{eta}`, `{elapsed}`, `{speed}` in labels.
- **Colors:** Pass color names or hex codes (`'#ffaa00'`) for bars, spinners, and text.
- **CSS/HTML:** In notebooks, customize with inline CSS and SVG for advanced visuals.
- **Thread control:** Pause, resume, stop, and restart bars/spinners with `BarThreadManager`.

---

## Modules Overview

- **progress_bar.py:** Terminal progress bars (single, nested, parallel).
- **spinner.py:** Terminal spinners with color and threading.
- **ntbk_progbar.py:** Notebook progress bars with HTML/CSS styling.
- **ntbk_spinner.py:** Notebook spinners with advanced animation.
- **utils.py:** Color handling, terminal control, thread management, error classes.
- **manager.py:** Assets manager for progressive_py

---

## Developed by **Muhammad Abubakar Siddique Ansari**
