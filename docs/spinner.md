# progressive_py Spinner Module

This module provides animated spinners for terminal and notebook environments. Spinners are useful for indicating ongoing work, especially for tasks where progress cannot be easily quantified. The module supports rich color customization, threading, and integration with progress bars.

---

## Classes

### Spinner

Animated spinner class for terminal and notebook output.

**Initialization Arguments (`args` dictionary or keyword arguments):**
- `text`: Text to display alongside the spinner (default `"Loading"`)
- `spn_side`: Position of spinner relative to text (`"right"` or `"left"`, default `"right"`)
- `line`: Vertical position for stacking multiple spinners (default `0`)
- `seq`: Sequence of spinner characters (default `['|', '/', '-', '\\']`)
- `interval`: Delay between spinner frames in seconds (default `0.05`)
- `final_text`: Text to display when spinner stops (default `text + '...Done!'`)
- `final_clear`: Whether to clear spinner line on finish (default `True`)
- `fg_text`: List of foreground colors for text (default `['white']`)
- `bg_text`: List of background colors for text (default `['black']`)
- `fg_spnr`: List of foreground colors for spinner (default: uses `fg_text`)
- `bg_spnr`: List of background colors for spinner (default: uses `bg_text`)
- `clr_interval`: List of intervals for color cycling (default `[interval]`)

**Key Methods:**
- `start()`: Starts the spinner animation in a background thread.
- `stop()`: Stops the spinner and displays the final text.
- `pause()`: Pauses the spinner animation.
- `resume()`: Resumes the spinner animation.
- `hide()`: Hides the spinner from output.
- `show()`: Shows the spinner.
- Context manager support (`__enter__`, `__exit__`).

---

## Functions

### spinning_work(args={}, work=lambda: time.sleep(10))

Runs a spinner while executing a given function.

**Arguments:**
- `args`: Spinner configuration dictionary.
- `work`: Function to execute while spinner runs. Receives the spinner instance as an argument.

**Returns:**  
`(spinner, output_of_work)`

**Usage Example:**
```python
def my_work(spinner):
    time.sleep(5)
spinning_work({"text": "Please wait..."}, my_work)
```

**Styled**
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

---

### work(pb, spn)

Example function for updating a progress bar and spinner together.

**Arguments:**
- `pb`: ProgressBar instance.
- `spn`: Spinner instance.

**Usage Example:**
```python
work(pb, spn)
```


---

### run_spinner_with_bar(spinner_args=None, bar_args=None, work=lambda pb, spn: work(pb, spn))

Runs a spinner and a hidden progress bar together, updating the spinner’s text with the progress bar’s output.

**Arguments:**
- `spinner_args`: Spinner configuration dictionary.
- `bar_args`: ProgressBar configuration dictionary.
- `work`: Function that receives both `pb` and `spn` and performs work.

**Returns:**  
`(ProgressBar, Spinner, output_of_work)`

**Usage Example:**
```python
from progressive_py.spinner import run_spinner_with_bar

def work(pb, spn):
    total = 100
    for i in range(total + 1):
        pb.update(i / total, i + 1, total)
        spn.text = "Processing:" + pb.bar_str.replace('\r', '')
        time.sleep(0.05)

run_spinner_with_bar({}, {}, work)
```

---

## Notes

- Spinners run in a separate thread for smooth animation.
- Supports color cycling for both spinner and text.
- Can be stacked vertically using the `line` argument.
- Integrates with progress bars for combined feedback.