# progressive_py Assets Manager Module

This module provides a flexible asset management system for storing, loading, renaming, and deleting JSON-based assets in categorized folders. It supports dynamic access, record keeping, and error handling for asset files.

---

## Classes

### AssetsManager

The main asset manager class.

**Initialization Arguments:**
- `package`: Python package name containing assets (default `'progressive_py'`)
- `assets_folder`: Folder inside the package for assets (default `'assets'`)
- `show_msg`: If `True`, prints status messages

---

## Methods

### save(content, category, file_key, name)
Saves a new asset or updates an existing one.

**Arguments:**
- `content`: Data to save (dict)
- `category`: Asset category (folder name)
- `file_key`: Asset file key (JSON file name, without extension)
- `name`: Asset name (key in JSON file)

### load(category, file_key, name)
Loads an asset by name.

**Arguments:**
- `category`: Asset category
- `file_key`: Asset file key
- `name`: Asset name

**Returns:** Asset data (dict)

### delete(category, file_key, name)
Deletes an asset from a file.

**Arguments:**
- `category`: Asset category
- `file_key`: Asset file key
- `name`: Asset name

### rename(category, file_key, old_name, new_name)
Renames an asset within a file.

**Arguments:**
- `category`: Asset category
- `file_key`: Asset file key
- `old_name`: Current asset name
- `new_name`: New asset name

### list_assets(category, file_key)
Lists all asset names in a file.

**Arguments:**
- `category`: Asset category
- `file_key`: Asset file key

**Returns:** List of asset names

### list_files(category)
Lists all asset files in a category.

**Arguments:**
- `category`: Asset category

**Returns:** List of file names

### list_categories()
Lists all asset categories.

**Returns:** List of category names

---

## Error Handling

### AssetNotFoundError

Raised when an asset or file is missing.

---

## Usage Example

```python
from progressive_py.manager import AssetsManager
from progressive_py.progress_bar import simple, time

mgr = AssetsManager()

# Load selected theme and style
theme = mgr.load('progress_bar', 'theme', 'neon_wave')
style = mgr.load('progress_bar', 'style', 'its_cool')

# Start stylish progress bar
for i in simple(
    range(100),
    dict(
        **theme,
        **style
    ),
    head='>',  # change parameters of passed dict before
    txt_lf="You're being hacked... {percent:.0f}% |",
    txt_rt='| ETA: {eta} | Elapsed: {elapsed}'
):
    time.sleep(0.05)
```

---

## Notes

- Assets are stored as JSON files under `assets/category/file_key.json` inside the package.
- Use `show_msg = True` to enable status messages.
- All asset management actions (save, load, rename, delete) prompt for user confirmation.
- Reading works after installation; writing only works in the source tree.