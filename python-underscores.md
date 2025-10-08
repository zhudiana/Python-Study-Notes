## Python Underscores: A Practical Guide

Python has a few naming conventions of using single or double underscore. These conventions allow us to:

- differentiate between public and non-public names in APIs.

- write subclasses safely.

- prevent name collisions and many more.

### 1) `_name` (single leading underscore)

Means **non-public** / internal implementation detail. Tools and humans treat it as “not part of the API,” but it’s still accessible.

```py
# shapes.py
_PI = 3.14159           # internal constant
def _normalize(v):      # internal helper
    return abs(v)
def area_circle(r):
    return _PI * r * r
```

`from module import *` skips names that start with \_ unless you explicitly include them in `__all__`. Other import forms can still access them, but that bypasses the convention

```py
__all__ = ["area_circle"]
```

### 2) `__name` (double leading underscore _in classes_)

Enables **name mangling** → `_ClassName__name`. Helps prevent accidental overrides in subclasses (not true privacy).

```py
class Base:
    def __init__(self): self.__token = "abc"  # becomes _Base__token
    def __calc(self): return 1

class Child(Base):
    def __calc(self): return 99  # becomes _Child__calc (no collision)
```

### 3) `name_` (single trailing underscore)

Use when a clean name clashes with a **keyword** or built-in.

```py
def from_(path): ...
class_ = "Economics"
```

### 4) `__dunder__` names

**Special hooks** defined by Python’s data model. Don’t invent your own; implement only documented ones (`__len__`, `__iter__`, `__enter__`, …).

```py
class Cart:
    def __len__(self): return 3

len(Cart())   # 3  (preferred)
```

### 5) Other common uses of `_` (recap)

- **Throwaway variable:** `for _ in range(n): ...`
- **Don’t-care in unpacking:** `a, _, c = tup` / `head, *_, tail = seq`
- **Digit separators:** `1_000_000`
- **Pattern-matching wildcard:** `case _:` (matches anything)
- **i18n alias:** `from gettext import gettext as _; _("Hi")`
- **REPL last result:** `_` is previous value (interactive only)

**Why this matters:**
Underscore conventions let you clearly separate **public APIs** from **implementation details**, avoid accidental subclass collisions, and keep code readable and intention-revealing.
