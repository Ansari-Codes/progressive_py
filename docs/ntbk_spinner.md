# progressive_py Notebook Spinner Module

This module provides animated, fully customizable spinners for Jupyter and IPython notebook environments. Spinners are rendered using HTML, CSS, and JavaScript, supporting advanced styling, color cycling, and integration with notebook progress bars.

---

## Classes

### NotebookDivSpinner

Animated spinner for notebook output, rendered as a styled HTML/CSS block.

**Initialization Arguments (`args` dictionary or keyword arguments):**
- `id`: Unique identifier for the spinner (auto-generated if not provided)
- `text`: Text to display alongside the spinner (default `"Loading"`)
- `final_text`: Text to display when spinner stops (default `"{text}... Done!"`)
- `spin_side`: Position of spinner relative to text (`"right"` or `"left"`, default `"right"`)
- `speed`: Animation speed (default `1.0`)
- `refresh`: Delay between spinner updates in seconds (default `0.05`)
- `fg_text`: List of foreground colors for text (default `"#6e6efc"`)
- `bg_text`: List of background colors for text (default `"transparent"`)
- `clr_interval`: List of intervals for color cycling (default `[0.5]`)
- `container_css`: Inline CSS for the spinner container
- `spinner_css`: Inline CSS for the spinner itself
- `add_css`: Additional CSS rules
- `text_css`: CSS for the text label
- `final_css`: CSS for the final text
- `spinner`: Dictionary for advanced spinner customization (e.g., SVG HTML, custom animation)

**Key Methods:**
- `start()`: Starts the spinner animation in a background thread.
- `stop()`: Stops the spinner and displays the final text.
- `pause()`: Pauses the spinner animation.
- `resume()`: Resumes the spinner animation.
- `hide()`: Hides the spinner from output.
- `show()`: Shows the spinner.
- `set_text(text)`: Updates the spinner’s text label live.
- Context manager support (`__enter__`, `__exit__`).

**Advanced Features:**
- **Color Cycling:**  
  Cycles through foreground/background colors at intervals for dynamic effects.
- **Custom Animation:**  
  Supports custom CSS keyframes and spinner HTML (SVG, etc.).
- **Live Updates:**  
  Uses JavaScript to update colors and text without re-rendering the whole spinner.

---

## Functions

### work(pb, spn)

Example function for updating a notebook progress bar and spinner together.

**Arguments:**
- `pb`: NotebookProgressBar instance.
- `spn`: NotebookDivSpinner instance.

**Usage Example:**
```python
for i in range(101):
    pb.update(i/100, i, 100)
    spn.set_text(pb.get_inline_html())
    time.sleep(0.025)
```

---

### run_spin_with_bar(spinner_args=None, bar_args=None, work=lambda pb, spn: work(pb, spn))

Runs a spinner and a hidden notebook progress bar together, updating the spinner’s text with the progress bar’s HTML.

**Arguments:**
- `spinner_args`: Spinner configuration dictionary.
- `bar_args`: NotebookProgressBar configuration dictionary.
- `work`: Function that receives both `pb` and `spn` and performs work.

**Returns:**  
`(NotebookProgressBar, NotebookDivSpinner)`

**Usage Example:**
```python
from progressive_py.ntbk_spinner import run_spin_with_bar

def work(pb, spn):
    for i in range(101):
        pb.update(i/100, i, 100)
        spn.set_text(pb.get_inline_html())
        time.sleep(0.025)

run_spin_with_bar({}, {}, work)
```

## One more example

```python
from progressive_py.ntbk_spinner import NotebookDivSpinner
import time

spinner = NotebookDivSpinner(
    text="🌈 Loading...",
    final_text="🎉 Done!",
    speed=1.2,
    refresh=0.1,

    fg_text=["#2afadf", "#00c9ff", "#ff0080", "#7928ca"],  # Rotating top border color
    bg_text=["#111111", "#222222", "#333333"],             # Rotating background & border

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

# Simulate loading
for i in range(21):
    spinner.set_text(f" <b>Loading...</b> {i*5}%")
    time.sleep(0.5)

spinner.stop()
```

---

## Advanced Styling & Customization

- **Custom Spinner HTML:**  
  Pass SVG or HTML via the `spinner_html` argument for unique spinner shapes.
- **CSS Animations:**  
  Customize animation speed, direction, and style via the `animation` argument.
- **Color Gradients:**  
  Cycle through multiple colors for both spinner and text.
- **Layout Control:**  
  Use `container_css` and `spinner_css` for full control over appearance.

---

## Notes

- Designed for Jupyter/IPython notebook environments.
- Uses `IPython.display` and JavaScript for live updates.
- Supports context manager usage for automatic cleanup.
- Integrates seamlessly with notebook progress bars for combined feedback.
- Fully customizable via HTML/CSS/SVG.
