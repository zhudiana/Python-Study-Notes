## Python Underscores

Python has a few naming conventions of using single or double underscore. These conventions allow us to:

- differentiate between public and non-public names in APIs.

- write subclasses safely.

- prevent name collisions and many more.

### 1) `_name` (single leading underscore)

Means **non-public** / internal implementation detail. Tools and humans treat it as “not part of the API,” but it’s still accessible.

```py
_PI = 3.14159           # internal constant
def _normalize(v):      # internal helper
    return abs(v)
def area_circle(r):
    return _PI * r * r
```

`from module import *` skips names that start with \_ unless you explicitly include them in `__all__`. Other import forms can still access them, but that bypasses the convention

```py
__all__ = ["_normalize", "area_circle"]
```

### 2) `__name` (double leading underscore _in classes_)

Enables **name mangling** → `_ClassName__name`. Helps prevent accidental overrides in subclasses (not true privacy).

```py
class Base:
    def __init__(self):
        self.__token = "abc"     # becomes _Base__token
    def __calc(self):
        return 1

class Child(Base):               # becomes _Child__calc (no collision)
    def __calc(self):
        return 99
```

### 3) `name_` (single trailing underscore)

Use this when a clean name would clash with a _keyword_ or a _built-in_. **Example**: `class_`, `id_`, `list_`. Keeps code readable without shadowing.

```py
def from_(path): ...
class_ = "Economics"        # avoids clashing with 'class' keyword
```

### 4) `__dunder__` names

**Special hooks** defined by Python’s data model. Don’t invent your own; implement only documented ones (`__len__`, `__iter__`, `__enter__`, …).

```py
class Cart:
    def __len__(self): return 3

len(Cart())     # 3  (preferred)
c.__len__()     # works but not idiomatic
```

### 5) Other common uses of `_` (recap)

- **Throwaway variable** : “I’m intentionally ignoring this value.”

```py
for _ in range(3):       # I don't need the loop index
    print("hi")
```

- **Digit separator in numbers** : “Make big numbers readable.”
  (Underscores don’t change the value.)

```py
population = 1_356_000_000     # same as 1356000000
pi_million = 3_141_592.0
```

- **Wildcard in `match`/`case`** : “Match anything; don’t bind it.”
  (PEP 634 structural pattern matching.)

```py
match value:
    case 0:
        print("zero")
    case _:
        print("something else")   # _ matches anything, no name created
```

**Why this matters:**
Underscore conventions let you clearly separate **public APIs** from **implementation details**, avoid accidental subclass collisions, and keep code readable and intention-revealing.
