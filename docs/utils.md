# progressive_py Utilities Module

This module provides essential utilities for the progressive_py library, including color handling, terminal control, error classes, and advanced thread management for running multiple progress bars or spinners concurrently.

---

## Color Utilities

- **rgb_fg(r, g, b):**  
  Returns ANSI escape code for foreground RGB color.

- **rgb_bg(r, g, b):**  
  Returns ANSI escape code for background RGB color.

- **hex_to_rgb(hex_color):**  
  Converts a hex color string (e.g., `#ffaa00`) to an RGB tuple.

- **rgb_to_hex(rgb):**  
  Converts an RGB tuple to a hex color string.

- **resolve_color(color_name, mode='fg'):**  
  Resolves a color name or hex code to an ANSI escape code for foreground (`'fg'`) or background (`'bg'`).  
  Uses `COLOR_MAP` and `BACKGROUND_COLOR_MAP` for named colors.

- **gradient_colors(start_color, end_color, steps):**  
  Generates a list of hex colors forming a gradient between two colors over a given number of steps.

---

## Terminal Control

- **move_cursor_up(n=1):**  
  Moves the terminal cursor up by `n` lines.

- **move_cursor_down(n=1):**  
  Moves the terminal cursor down by `n` lines.

- **clear_line():**  
  Clears the current terminal line and moves the cursor to the beginning.

- **oup:**  
  Alias for `sys.stdout`, used for output.

---

## Color Maps

- **COLOR_MAP:**  
  Maps color names (including bright and dim variants) to ANSI escape codes for foreground colors.

- **BACKGROUND_COLOR_MAP:**  
  Maps color names to ANSI escape codes for background colors.

---

## Error Classes

- **BarError(msg, *args):**  
  Custom exception for progress bar errors.

---

## Thread Management

### BarThreadManager

Manages and runs multiple progress bars or spinners concurrently using threads.

**Initialization:**
- Accepts a dictionary:  
  `{ "thread_name": [bar_instance, bar_task_function] }`

**Task Function Signature:**
```python
def task(bar, pause_event, stop_check, cool_down=0.05):
    # bar: ProgressBar or NotebookProgressBar instance
    # pause_event: threading.Event for pausing
    # stop_check: function to check for stop signal
    # cool_down: sleep interval between updates
```

**Key Methods:**
- `start(name, cool_down=0.05)`: Start a named thread.
- `pause(name)`: Pause a named thread.
- `resume(name)`: Resume a named thread.
- `stop(name)`: Stop a named thread.
- `restart(name, cool_down=0.05)`: Restart a named thread.
- `wait(name)`: Wait for a named thread to finish.
- `start_all(cool_down=0.05)`: Start all threads.
- `wait_all()`: Wait for all threads to finish.
- `pause_all()`: Pause all threads.
- `resume_all()`: Resume all threads.
- `stop_all()`: Stop all threads.
- `restart_all(cool_down=0.05)`: Restart all threads.

**Usage Example:**
```python
from progressive_py.progress_bar import ProgressBar
from progressive_py.utils import BarThreadManager

def example_task(bar, pause_event, stop_check, cool_down=0.05):
    total = 100
    for i in range(total):
        if stop_check(): break
        pause_event.wait()
        time.sleep(cool_down)
        bar.update(i / (total - 1), i + 1, total)

bar1 = ProgressBar({'txt_lf': '🚀 Task 1 {percent:.0f}% {eta} {elapsed}', 'line': 1})
bar2 = ProgressBar({'txt_lf': '🔧 Task 2 {percent:.0f}% {eta} {elapsed}', 'line': 0})

tasks = {
    "task1": [bar1, example_task],
    "task2": [bar2, example_task],
}

manager = BarThreadManager(tasks)
manager.start_all(cool_down=0.02)
manager.wait_all()
print("\n✅ All tasks completed.")
```

---

## Miscellaneous

- **fast_unique_id(prefix='', suffix=''):**  
  Generates a fast, unique string ID using the current time and a random number. Useful for assigning unique IDs to bars and spinners.

---

## Notes

- All color handling is compatible with both terminal and notebook environments.
- Thread management supports pausing, resuming, and stopping tasks for advanced workflows.
- Custom exceptions help with error handling in progress bar logic.
