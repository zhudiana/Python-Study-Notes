## Python Asterisk (\*) in Function Parameters

When working with Python functions, you'll encounter the asterisk (`*`) used in ways beyond multiplication. The bare asterisk and `*args` are special parameters that control **how arguments are passed** to functions - specifically, whether they must be passed by **position**, by **keyword**, or **both**.

## What is the Bare Asterisk (`*`)?

The bare asterisk in a function definition acts as a **boundary marker** that forces all parameters after it to be passed as **keyword-only arguments**.

### Basic Syntax

```python
def function_name(param1, *, param2, param3):
    # param1 can be positional or keyword
    # param2 and param3 MUST be keyword-only
    pass
```

### Why Use It?

1. **Explicit Code** - Forces callers to use descriptive parameter names
2. **Prevents Errors** - Reduces mistakes from positional argument confusion
3. **Better Readability** - Makes function calls self-documenting
4. **API Stability** - Allows changing parameter order without breaking code

## Practical Examples

### Example 1: Keyword-Only Parameters

```python
def create_user(username, *, email, age, role="user"):
    """
    username: can be passed positionally or by keyword
    email, age, role: MUST be passed by keyword
    """
    return {
        "username": username,
        "email": email,
        "age": age,
        "role": role
    }

# ✅ Valid calls
create_user("john_doe", email="john@example.com", age=25)
create_user(username="john_doe", email="john@example.com", age=25)

# ❌ Invalid - email and age must be keyword arguments
create_user("john_doe", "john@example.com", 25)
# TypeError: create_user() takes 1 positional argument but 3 were given
```

### Example 2: All Parameters Keyword-Only

Place the asterisk **first** to require all arguments as keywords:

```python
def configure_server(*, host, port, timeout=30, ssl=True):
    """All parameters must be passed by keyword"""
    return f"Server: {host}:{port} (SSL: {ssl}, Timeout: {timeout}s)"

# ✅ Valid - all by keyword
configure_server(host="localhost", port=8080)
configure_server(host="localhost", port=8080, ssl=False, timeout=60)

# ❌ Invalid - no positional arguments allowed
configure_server("localhost", 8080)
# TypeError: configure_server() takes 0 positional arguments but 2 were given
```

## `*args` vs Bare Asterisk (`*`)

### `*args` - Collects Extra Positional Arguments

```python
def calculate_average(*args):
    """Accepts any number of positional arguments"""
    return sum(args) / len(args) if args else 0

calculate_average(1, 2, 3)        # 2.0
calculate_average(10, 20, 30, 40) # 25.0
```

### Combining `*args` with Keyword-Only Parameters

```python
def log_message(level, *messages, timestamp=None, user=None):
    """
    level: positional
    *messages: any number of additional positional args
    timestamp, user: keyword-only (after *messages)
    """
    full_message = f"[{level}] {' '.join(messages)}"
    if user:
        full_message += f" (User: {user})"
    return full_message

# ✅ Valid usage
log_message("ERROR", "Database", "connection", "failed", user="admin")
# Output: "[ERROR] Database connection failed (User: admin)"

# ❌ Invalid - timestamp and user must be keywords
log_message("ERROR", "Failed", "admin", "2024-01-01")
```

---

## Summary

```python
def example_function(pos1, pos2, /, pos_or_kw, *, kw1, kw2):
    #                 pos1, pos2: positional-only (before /)
    #                            pos_or_kw: flexible (between / and *)
    #                                       kw1, kw2: keyword-only (after *)
    pass
```

**Parameter Types:**

- **Before `/`** → Positional-only
- **Between `/` and `*`** → Positional or keyword
- **After `*`** → Keyword-only

---

## Common Patterns and Best Practices

### Pattern 1: Mixed Flexibility

```python
def process_data(data, *, validation=True, timeout=30):
    """
    data: simple, can be positional
    validation, timeout: explicit options, keyword-only
    """
    pass

process_data([1, 2, 3], validation=False)  # Clean and clear
```

### Pattern 2: API Functions

```python
def api_request(endpoint, *, method="GET", headers=None, timeout=30):
    """
    Common in web frameworks (FastAPI, Flask)
    endpoint: required, position makes sense
    method, headers, timeout: explicit configuration
    """
    pass
```

### Pattern 3: Database Operations

```python
def query_database(table, filters, *,
                   order_by=None,
                   limit=100,
                   offset=0,
                   include_deleted=False):
    """
    table, filters: core requirements
    order_by, limit, offset, include_deleted: explicit options
    """
    pass
```

## Important Rules

1. **Asterisk Cannot Be Last**

   ```python
   # ❌ SyntaxError
   def bad_function(param1, param2, *):
       pass
   ```

2. **Asterisk Cannot Appear Twice**

   ```python
   # ❌ SyntaxError
   def bad_function(param1, *args, *, param2):
       pass
   ```

3. **Positional Args Before Keyword Args**

   ```python
   # ❌ SyntaxError
   function_call(keyword="value", "positional")

   # ✅ Correct
   function_call("positional", keyword="value")
   ```

## When to Use Keyword-Only Parameters

**Use when:**

- Parameters are configuration options (flags, timeouts, modes)
- Parameter names clarify intent (`include_deleted=True` vs `True`)
- You want to prevent positional argument mistakes
- Building public APIs that might evolve
- Working with many optional parameters

**Avoid when:**

- There are very few parameters (1-2)
- Parameter order is completely obvious
- Adding unnecessary verbosity to simple functions

## Quick Reference

```python
# Regular function - all flexible
def func(a, b, c):
    pass
func(1, 2, 3)                    # ✅
func(a=1, b=2, c=3)              # ✅

# Keyword-only after asterisk
def func(a, *, b, c):
    pass
func(1, b=2, c=3)                # ✅
func(1, 2, 3)                    # ❌

# All keyword-only
def func(*, a, b, c):
    pass
func(a=1, b=2, c=3)              # ✅
func(1, 2, 3)                    # ❌

# With *args and keyword-only
def func(a, *args, b, c):
    pass
func(1, 2, 3, b=4, c=5)          # ✅
# args will be (2, 3)
```

## Resources

- [PEP 3102 - Keyword-Only Arguments](https://peps.python.org/pep-3102/)
- [Real Python - Python Asterisk and Slash Special Parameters](https://realpython.com/python-asterisk-and-slash-special-parameters/)
- [Python Documentation - Function Definitions](https://docs.python.org/3/reference/compound_stmts.html#function-definitions)
