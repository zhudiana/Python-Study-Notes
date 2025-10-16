# Python Underscores: A Complete Guide

Python uses underscores (`_`) in variable and function names to communicate intent and control visibility. Understanding these conventions helps you write better, more maintainable code.

---

## Table of Contents

1. [Single Leading Underscore (`_name`)](#1-single-leading-underscore-_name)
2. [Double Leading Underscore (`__name`)](#2-double-leading-underscore-__name)
3. [Single Trailing Underscore (`name_`)](#3-single-trailing-underscore-name_)
4. [Double Leading and Trailing (`__dunder__`)](#4-double-leading-and-trailing-__dunder__)
5. [Special Uses of `_`](#5-special-uses-of-_)

---

## 1. Single Leading Underscore (`_name`)

### What It Means

**"This is internal/private - not part of the public API"**

This is a **convention**, not enforced by Python. It tells other developers (and tools) that this name is an implementation detail that might change.

### When to Use

- Internal helper functions
- Private module constants
- Implementation details you don't want users to depend on

### Example

```python
# math_utils.py
_PI = 3.14159                    # Internal constant

def _normalize(value):           # Internal helper function
    """Private helper - not for external use"""
    return abs(value)

def calculate_area(radius):      # Public API
    """Calculate circle area - safe to use"""
    return _PI * radius * radius
```

### How It Affects Imports

**Wildcard imports (`from module import *`) skip names starting with `_`:**

```python
# user.py
from math_utils import *

print(calculate_area(5))    # ✅ Works - public function
print(_PI)                  # ❌ NameError - not imported
print(_normalize(-10))      # ❌ NameError - not imported
```

**Direct imports still work (but you're breaking the convention):**

```python
from math_utils import _PI, _normalize   # ✅ Works, but discouraged

print(_PI)                  # 3.14159 - works but you shouldn't rely on this
```

### Controlling Wildcard Imports with `__all__`

```python
# math_utils.py
__all__ = ["calculate_area"]  # Only export this

_PI = 3.14159
def _normalize(value):
    return abs(value)
def calculate_area(radius):
    return _PI * radius * radius
```

Now `from math_utils import *` only imports `calculate_area`.

### Real-World Example

```python
# database.py
_connection_pool = []            # Internal implementation detail

def _get_connection():           # Internal helper
    """Get connection from pool"""
    return _connection_pool.pop() if _connection_pool else create_new()

def query_database(sql):         # Public API
    """Execute SQL query - safe to use"""
    conn = _get_connection()
    return conn.execute(sql)
```

**Key Takeaway:** `_name` says "I'm internal - use at your own risk!"

---

## 2. Double Leading Underscore (`__name`)

### What It Means

**"Mangle this name to avoid accidental overrides in subclasses"**

Python automatically renames `__name` to `_ClassName__name`. This is called **name mangling**.

### When to Use

- When you want to prevent subclasses from accidentally overriding your attributes/methods
- Creating truly private implementation details in classes
- **Note:** This is NOT about security - it's about preventing naming collisions

### How Name Mangling Works

```python
class BankAccount:
    def __init__(self):
        self.__balance = 0       # Becomes _BankAccount__balance

    def __validate(self):        # Becomes _BankAccount__validate
        return self.__balance >= 0

account = BankAccount()
print(account.__balance)         # ❌ AttributeError
print(account._BankAccount__balance)  # ✅ Works (but don't do this!)
```

### Why It's Useful: Preventing Subclass Collisions

```python
class Base:
    def __init__(self):
        self.__secret = "base_secret"      # → _Base__secret

    def __calculate(self):                 # → _Base__calculate
        return 100

    def public_method(self):
        return self.__calculate()          # Calls _Base__calculate

class Child(Base):
    def __init__(self):
        super().__init__()
        self.__secret = "child_secret"     # → _Child__secret (different!)

    def __calculate(self):                 # → _Child__calculate (different!)
        return 200

    def child_method(self):
        return self.__calculate()          # Calls _Child__calculate

# Usage
obj = Child()
print(obj.public_method())    # 100 (calls Base's __calculate)
print(obj.child_method())     # 200 (calls Child's __calculate)
print(obj._Base__secret)      # "base_secret"
print(obj._Child__secret)     # "child_secret"
```

**Without name mangling, they would collide!**

### Real-World Example: Payment Processing

```python
class PaymentProcessor:
    def __init__(self):
        self.__api_key = "secret_key_123"     # Can't be accidentally overridden

    def __encrypt(self, data):                # Internal encryption logic
        return f"encrypted_{data}"

    def process_payment(self, amount):
        encrypted = self.__encrypt(amount)
        return f"Processing {encrypted} with key {self.__api_key}"

class StripeProcessor(PaymentProcessor):
    def __init__(self):
        super().__init__()
        self.__api_key = "stripe_key_456"     # Different from parent's __api_key!

    def __encrypt(self, data):                # Doesn't override parent's __encrypt
        return f"stripe_encrypted_{data}"

processor = StripeProcessor()
print(processor.process_payment(100))
# Output: Processing encrypted_100 with key secret_key_123
# (Uses parent's __api_key and __encrypt, not child's)
```

### When NOT to Use

```python
class User:
    def __init__(self, name):
        self.__name = name    # ❌ Overkill - just use self._name
```

Most of the time, `_name` (single underscore) is enough. Use `__name` only when you need protection from subclass collisions.

**Key Takeaway:** `__name` prevents accidental naming conflicts in inheritance hierarchies.

---

## 3. Single Trailing Underscore (`name_`)

### What It Means

**"I wanted to use this name, but it's a Python keyword"**

Use this convention to avoid conflicts with reserved keywords or built-in names.

### Python Keywords You Can't Use

```python
# These are ALL invalid:
class = "Math"        # ❌ SyntaxError
def = 5               # ❌ SyntaxError
for = [1, 2, 3]       # ❌ SyntaxError
if = True             # ❌ SyntaxError
```

### Solution: Add Trailing Underscore

```python
class_ = "Math"       # ✅ Works
def_ = 5              # ✅ Works
for_ = [1, 2, 3]      # ✅ Works
if_ = True            # ✅ Works
```

### Common Use Cases

#### 1. Avoiding Keyword Conflicts

```python
def create_class(name, class_):    # 'class' is a keyword, so use 'class_'
    return f"Creating {class_} named {name}"

create_class("Python 101", class_="beginner")
```

#### 2. Avoiding Built-in Shadowing

```python
# ❌ Bad - shadows built-in functions
def process_data(list, dict, id):
    pass

# ✅ Good - preserves built-in names
def process_data(list_, dict_, id_):
    items = list(range(10))    # Can still use built-in list()
    return dict_(zip(list_, id_))
```

#### 3. Function Parameter Names

```python
from datetime import datetime

def schedule_task(from_, to_, type_):
    """
    from_ : avoid 'from' keyword
    to_   : avoid 'to' keyword (rare, but consistent)
    type_ : avoid shadowing built-in type()
    """
    return f"Task from {from_} to {to_} of type {type_}"

schedule_task(
    from_=datetime.now(),
    to_=datetime.now(),
    type_="email"
)
```

### Real-World Examples

#### Django Models

```python
class User(models.Model):
    class_ = models.CharField(max_length=50)    # 'class' is a keyword
    type_ = models.CharField(max_length=20)     # Avoid shadowing type()
```

#### SQLAlchemy

```python
from sqlalchemy import Column, String

class Student(Base):
    __tablename__ = 'students'

    id_ = Column('id', String, primary_key=True)     # 'id' shadows built-in
    class_ = Column('class', String)                 # 'class' is keyword
```

**Key Takeaway:** `name_` keeps your code clean when the perfect name is a keyword.

---

## 4. Double Leading and Trailing (`__dunder__`)

### What It Means

**"This is a special method defined by Python's data model"**

These are called **"magic methods"** or **"dunder methods"** (double underscore). Python calls them automatically in response to specific operations.

### Rules

1. **Only use documented dunder methods** - don't invent your own
2. **Python calls them, not you** (usually)
3. They enable operator overloading and protocol implementation

### Common Dunder Methods

#### Object Lifecycle

```python
class Database:
    def __init__(self, name):        # Constructor
        print(f"Creating database: {name}")
        self.name = name

    def __del__(self):               # Destructor
        print(f"Closing database: {self.name}")

db = Database("mydb")    # Prints: Creating database: mydb
del db                   # Prints: Closing database: mydb
```

#### String Representation

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __str__(self):               # For print() and str()
        return f"{self.title} by {self.author}"

    def __repr__(self):              # For debugging and repr()
        return f"Book(title='{self.title}', author='{self.author}')"

book = Book("1984", "Orwell")
print(book)          # 1984 by Orwell (calls __str__)
print(repr(book))    # Book(title='1984', author='Orwell') (calls __repr__)
```

#### Comparison Operators

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def __eq__(self, other):         # ==
        return self.celsius == other.celsius

    def __lt__(self, other):         # <
        return self.celsius < other.celsius

    def __le__(self, other):         # <=
        return self.celsius <= other.celsius

temp1 = Temperature(20)
temp2 = Temperature(25)
print(temp1 < temp2)     # True (calls __lt__)
print(temp1 == temp2)    # False (calls __eq__)
```

#### Arithmetic Operators

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):        # +
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):       # *
        return Vector(self.x * scalar, self.y * scalar)

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)       # Vector(4, 6)
print(v1 * 3)        # Vector(3, 6)
```

#### Container Protocol

```python
class Playlist:
    def __init__(self):
        self.songs = []

    def __len__(self):               # len()
        return len(self.songs)

    def __getitem__(self, index):    # playlist[0]
        return self.songs[index]

    def __setitem__(self, index, value):  # playlist[0] = "Song"
        self.songs[index] = value

    def __contains__(self, song):    # "Song" in playlist
        return song in self.songs

playlist = Playlist()
playlist.songs = ["Song1", "Song2", "Song3"]
print(len(playlist))           # 3
print(playlist[0])             # Song1
print("Song2" in playlist)     # True
```

#### Context Manager (with statement)

```python
class FileManager:
    def __init__(self, filename):
        self.filename = filename

    def __enter__(self):             # Called when entering 'with' block
        print(f"Opening {self.filename}")
        self.file = open(self.filename, 'w')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):  # Called when exiting
        print(f"Closing {self.filename}")
        self.file.close()

with FileManager("test.txt") as f:
    f.write("Hello World")
# Automatically calls __exit__ here
```

#### Iterator Protocol

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):              # Makes object iterable
        return self

    def __next__(self):              # Called on each iteration
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

for num in Countdown(5):
    print(num)          # 5, 4, 3, 2, 1
```

#### Callable Objects

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, x):           # Makes object callable like a function
        return x * self.factor

double = Multiplier(2)
triple = Multiplier(3)
print(double(5))        # 10
print(triple(5))        # 15
```

### Complete List of Common Dunders

| Category         | Methods                                                    | Purpose                 |
| ---------------- | ---------------------------------------------------------- | ----------------------- |
| **Construction** | `__init__`, `__new__`, `__del__`                           | Create/destroy objects  |
| **String**       | `__str__`, `__repr__`, `__format__`                        | String representations  |
| **Comparison**   | `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__` | Comparison operators    |
| **Arithmetic**   | `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__mod__`  | Math operators          |
| **Container**    | `__len__`, `__getitem__`, `__setitem__`, `__contains__`    | List-like behavior      |
| **Context**      | `__enter__`, `__exit__`                                    | Context managers (with) |
| **Iterator**     | `__iter__`, `__next__`                                     | For loops               |
| **Callable**     | `__call__`                                                 | Make objects callable   |
| **Attribute**    | `__getattr__`, `__setattr__`, `__delattr__`                | Attribute access        |

### DON'T Invent Your Own!

```python
# ❌ BAD - Don't create custom dunders
class MyClass:
    def __my_method__(self):    # Don't do this!
        pass

# ✅ GOOD - Use normal names or single underscore
class MyClass:
    def _my_method(self):       # Internal method
        pass
```

**Key Takeaway:** Dunder methods let Python call your code automatically. Only use documented ones!

---

## 5. Special Uses of `_`

The underscore has additional special meanings in different contexts.

### 5.1 Throwaway Variable

When you don't care about a value, use `_` to signal "I'm intentionally ignoring this."

#### Examples

```python
# Loop without using the index
for _ in range(5):
    print("Hello")

# Unpacking when you don't need all values
name, _, age = ("John", "unused_middle", 25)
print(f"{name} is {age}")    # John is 25

# Ignoring multiple values
first, *_, last = [1, 2, 3, 4, 5]
print(first, last)    # 1 5

# Function returns you don't need
result, _ = divmod(10, 3)
print(result)         # 3 (quotient only)

# Catching exceptions you don't use
try:
    risky_operation()
except ValueError as _:
    print("Error occurred")    # Don't need the exception object
```

#### In List Comprehensions

```python
# Create list of 10 empty lists
matrix = [[] for _ in range(10)]

# Execute function 5 times
results = [process() for _ in range(5)]
```

### 5.2 Number Separators (Python 3.6+)

Use underscores to make large numbers more readable. They're completely ignored by Python.

#### Examples

```python
# Large integers
population = 1_356_000_000        # 1.356 billion
distance = 299_792_458            # Speed of light (m/s)

# Binary, Octal, Hex
binary = 0b_1111_0000_1010        # Group binary digits
hex_color = 0xFF_AA_00            # RGB color

# Floats
pi = 3.141_592_653_589_793
planck = 6.626_070_15e-34

# You can place them anywhere (except start/end or around decimal point)
valid = 1_0_0_0                   # Valid but ugly
invalid = _100                    # ❌ SyntaxError
invalid = 100_                    # ❌ SyntaxError
invalid = 3._14                   # ❌ SyntaxError
```

#### Real-World Usage

```python
# Financial calculations
SALARY = 75_000
BONUS = 10_000
TOTAL_COMPENSATION = SALARY + BONUS    # 85000

# Physics constants
SPEED_OF_LIGHT = 299_792_458           # m/s
AVOGADRO = 6.022_140_76e23            # particles/mol

# Byte sizes
ONE_MEGABYTE = 1_048_576              # 1024 * 1024
ONE_GIGABYTE = 1_073_741_824          # 1024³
```

### 5.3 Wildcard in Pattern Matching (Python 3.10+)

In `match`/`case` statements, `_` matches anything without binding it to a variable.

#### Basic Example

```python
def classify_number(num):
    match num:
        case 0:
            return "zero"
        case 1 | 2 | 3:
            return "small"
        case _:                # Matches anything else
            return "other"

print(classify_number(0))      # "zero"
print(classify_number(2))      # "small"
print(classify_number(100))    # "other"
```

#### Advanced Pattern Matching

```python
def handle_command(command):
    match command:
        case ["quit"]:
            return "Exiting"

        case ["load", filename]:
            return f"Loading {filename}"

        case ["save", filename, _]:      # Ignore third argument
            return f"Saving {filename}"

        case ["move", x, y]:
            return f"Moving to ({x}, {y})"

        case _:                           # Default case
            return "Unknown command"

print(handle_command(["load", "file.txt"]))           # Loading file.txt
print(handle_command(["save", "data.db", "backup"]))  # Saving data.db
print(handle_command(["invalid"]))                     # Unknown command
```

#### Pattern Matching with Objects

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

def describe_point(point):
    match point:
        case Point(x=0, y=0):
            return "Origin"
        case Point(x=0, y=_):              # x is 0, don't care about y
            return "On Y-axis"
        case Point(x=_, y=0):              # y is 0, don't care about x
            return "On X-axis"
        case Point(x=x, y=y) if x == y:
            return "On diagonal"
        case _:
            return "Somewhere else"

print(describe_point(Point(0, 0)))     # Origin
print(describe_point(Point(0, 5)))     # On Y-axis
print(describe_point(Point(3, 0)))     # On X-axis
```

### 5.4 Interactive Interpreter

In Python's interactive shell, `_` holds the result of the last expression.

```python
>>> 2 + 2
4
>>> _          # References previous result
4
>>> _ * 3
12
>>> _
12
```

### 5.5 Internationalization (i18n)

By convention, `_()` is used as an alias for translation functions.

```python
import gettext
_ = gettext.gettext

print(_("Hello, World!"))    # Translates "Hello, World!" to user's language
```

---

## Summary Table

| Pattern        | Meaning                                  | Example                    |
| -------------- | ---------------------------------------- | -------------------------- |
| `_name`        | Internal/private (convention)            | `_internal_helper()`       |
| `__name`       | Name mangling (avoid subclass collision) | `self.__private_var`       |
| `name_`        | Avoid keyword conflict                   | `class_`, `from_`, `type_` |
| `__dunder__`   | Special Python method                    | `__init__`, `__str__`      |
| `_` alone      | Throwaway variable                       | `for _ in range(10)`       |
| `_` in numbers | Digit separator                          | `1_000_000`                |
| `_` in match   | Wildcard pattern                         | `case _:`                  |
| `_` in REPL    | Last result                              | `>>> _`                    |

---

## Best Practices

### ✅ DO

```python
# Use single underscore for internal stuff
def _helper_function():
    pass

# Use double underscore when you need name mangling
class Base:
    def __init__(self):
        self.__private_data = []

# Use trailing underscore for keyword conflicts
def process(class_, type_):
    pass

# Use underscore for throwaway variables
for _ in range(5):
    do_something()

# Use underscores in large numbers
price = 1_999_999
```

### ❌ DON'T

```python
# Don't use double underscore everywhere
class User:
    def __init__(self):
        self.__name = "John"    # ❌ Overkill, use self._name

# Don't invent your own dunders
def __my_function__():          # ❌ Reserved for Python
    pass

# Don't access mangled names directly
obj._ClassName__private         # ❌ Defeats the purpose

# Don't use single underscore for public APIs
def _public_function():         # ❌ Confusing - looks private
    """This is meant to be used by everyone"""
    pass
```

---

## Why This Matters

Understanding underscore conventions helps you:

1. **Write clearer code** - Signal your intentions to other developers
2. **Avoid naming conflicts** - Use `name_` for keywords, `__name` for inheritance
3. **Design better APIs** - Mark internal details with `_name`
4. **Understand Python's magic** - Know how `__dunder__` methods work
5. **Write idiomatic Python** - Follow community standards

---

## Resources

- [PEP 8 - Style Guide](https://peps.python.org/pep-0008/)
- [PEP 634 - Pattern Matching](https://peps.python.org/pep-0634/)
- [Python Data Model Documentation](https://docs.python.org/3/reference/datamodel.html)
- [Real Python - Underscores in Python](https://realpython.com/python-double-underscore/)

---

**Master these conventions, and your Python code will be cleaner, safer, and more professional! 🐍**
