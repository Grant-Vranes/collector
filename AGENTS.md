@ -0,0 +1,256 @@
# AGENTS.md - Guidelines for AI Coding Agents

## Project Overview

Collector is a GNOME desktop application for drag-and-drop file management, built with Python, GTK 4, and LibAdwaita. It uses the Meson build system and is distributed as a Flatpak.

## Build Commands

### Setup and Build (Meson)

```bash
# Configure build
meson setup _build

# Build the project
meson compile -C _build

# Install locally (for testing)
meson install -C _build

# Clean build
rm -rf _build && meson setup _build
```

### Flatpak Development

```bash
# Build Flatpak
flatpak-builder --user --install _flatpak it.mijorus.collector.json

# Run the app
flatpak run it.mijorus.collector

# Run with debug logging
flatpak run --env=APP_DEBUG=1 it.mijorus.collector

# Run with multiple windows
flatpak run it.mijorus.collector --w=3
```

### Linting and Type Checking

```bash
# Type checking with pyright
pyright src/

# Or with specific config
pyright --config pyrightconfig.json
```

### Testing

No test suite exists in this project. When adding tests, place them in a `tests/` directory at the project root.

## Code Style Guidelines

### Imports

```python
# Standard library imports first (alphabetically)
import csv
import logging
import os
from typing import Optional

# Third-party imports (PyGObject gi.require_version before imports)
import gi
gi.require_version('Gtk', '4.0')
gi.require_version('Adw', '1')

from gi.repository import Gtk, Adw, Gio, Gdk, GLib

# Local imports last (relative)
from .lib.constants import APP_ID
from .lib.utils import get_gsettings
```

### File Header

All Python source files include the GPL license header:

```python
# filename.py
#
# Copyright 2023 lorenzo
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
# SPDX-License-Identifier: GPL-3.0-or-later
```

### Naming Conventions

- **Classes**: PascalCase (e.g., `CollectorWindow`, `DroppedItem`)
- **Functions/Methods**: snake_case (e.g., `get_gsettings`, `on_drop_event`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `APP_ID`, `SUPPORTED_IMG_TYPES`)
- **Class attributes**: UPPER_SNAKE_CASE for constants, snake_case for instance variables
- **Private methods**: Prefix with underscore (e.g., `_private_method`)
- **Signal handlers**: Prefix with `on_` (e.g., `on_click_open_uri`, `on_drop_event`)

### Type Annotations

Use type hints for function signatures:

```python
def get_file_hash(file: Gio.File, alg: str = 'md5') -> str:
    ...

def link_is_image(link: str) -> tuple[bool, str]:
    ...

class CarouselItem:
    def __init__(self, item: DroppedItem, image: Gtk.Image, index: int):
        ...
```

Use `Optional` for nullable types:

```python
self.window_color_btn: Optional[Gtk.Button] = None
```

### Error Handling

Define custom exceptions for domain-specific errors:

```python
class DroppedItemNotSupportedException(Exception):
    def __init__(self, item, msg, *args: object) -> None:
        super().__init__(*args)
        self.item = item
        logging.warn(msg)
```

Use logging instead of print statements:

```python
logging.debug(f'Creating item from type: {type(item)}')
logging.warn('Warning message')
logging.info('Info message')
```

### GTK/UI Patterns

**Widget creation with CSS classes:**

```python
button = Gtk.Button(
    css_classes=['circular', 'opaque'],
    icon_name='plus-symbolic',
    valign=Gtk.Align.CENTER,
)
```

**Template classes:**

```python
@Gtk.Template(resource_path='/it/mijorus/collector/gtk/preferences.ui')
class SettingsWindow(Adw.PreferencesWindow):
    __gtype_name__ = "SettingsWindow"
    
    keep_items_when_dragging = Gtk.Template.Child()
```

**GSettings bindings:**

```python
self.settings.bind('keep-on-drag', self.keep_items_when_dragging,
                   'active', Gio.SettingsBindFlags.DEFAULT)
```

**Signal connections:**

```python
button.connect('clicked', self.on_button_clicked)
settings.connect('changed::keep-on-drag', self.on_setting_changed)
```

### File Structure

```
src/
├── __init__.py
├── main.py           # Application entry point and main class
├── window.py         # Main application window
├── preferences.py    # Settings/preferences window
├── collector.gresource.xml
├── collector.in      # Entry point template
├── lib/              # Utility modules
│   ├── __init__.py
│   ├── constants.py  # Application constants
│   ├── utils.py      # Utility functions
│   ├── DroppedItem.py
│   ├── CarouselItem.py
│   └── CsvCollector.py
├── gtk/              # UI template files (.ui)
└── assets/           # CSS and static assets
```

### GSettings Schema

Settings are stored in `data/it.mijorus.collector.gschema.xml`. Access via:

```python
from .lib.utils import get_gsettings
settings = get_gsettings()
value = settings.get_boolean('keep-on-drag')
```

### Internationalization

Use `_()` for translatable strings:

```python
EMPTY_DROP_TEXT = _('Drop content here')
translator_credits = _("translator-credits")
```

Update translations with:

```bash
# Regenerate .pot file
xgettext -o po/collector.pot src/*.py

# Update .po files
msgmerge po/es.po po/collector.pot -o po/es.po
```

## Key Dependencies

- **GTK 4.0** and **Adw 1** - GUI framework
- **PyGObject** - Python bindings for GObject
- **Pillow** - Image processing
- **requests** - HTTP requests for image downloading
- **pybind11** - C++ bindings (for native extensions)

## Debugging

Enable debug logging with environment variable:

```bash
APP_DEBUG=1 flatpak run it.mijorus.collector
```

Log files are stored at: `$XDG_CACHE_DIR/logs/collector.log`