# Don’t Use Bare `except:` in Python (and What To Do Instead)

Catching errors is good. **Catching _everything_** is not.

This note explains:

- What `try/except` does in Python
- What a **bare `except:`** is
- Why it’s risky (with real examples)
- What to do instead (specific exceptions, `Exception` as a safety net, and app-level handlers)
- How this plays out in **FastAPI / Pydantic**

---

## 1) `try / except`

```python
try:
    risky_operation()
except ValueError:
    fix_it()
```

- Code inside `try:` may raise errors (exceptions).
- `except SomeError:` handles a specific error type.

You can also add `else:` (runs if no exception) and `finally:` (always runs).

## 2) What is **bare `except:`**?

A **bare except** is an `except` with **no type**:

```python
try:
    do_thing()
except:
    pass
```

This means **“catch literally everything.”** In Python, that includes not just normal errors but also special signals like `KeyboardInterrupt` (Ctrl+C) and `SystemExit`.

## 3) Why bare `except:` is dangerous

### a) It catches things you should _never_ swallow

```python
try:
    long_running_task()
except:
    pass  # also eats KeyboardInterrupt / SystemExit
```

You press Ctrl+C to stop the program… **nothing happens**. Your app refuses to die and that is not fun.

### b) It hides real bugs (“fail silently”)

```python
def parse_price(s):
    try:
        return float(s)
    except:
        return 0.0  # masking every possible bug with a fake value
```

Now a `TypeError`, `AttributeError`, _or_ `ValueError` all return `0.0`. Good luck debugging wrong totals later.

### c) It turns debugging into guesswork

```python
try:
    update_db()
except:
    logger.error("something failed")  # no stacktrace, no error type
```

No error type, no traceback → you can’t reproduce or fix the problem.

## 4) What to do instead (the safe patterns)

### Catch **known** exceptions first

```python
try:
    user = json.loads(body)
    age = int(user["age"])
except (json.JSONDecodeError, KeyError, ValueError) as e:
    return {"error": f"bad input: {e}"}
```

Be explicit. You’re telling future-you and your teammates which failures you expect.

### Use `except Exception as e` as a **final safety net** (not bare)

```python
try:
    process()
except KnownError as e:
    handle(e)
except Exception as e:                 # safety net
    logger.exception("Unexpected error")
    raise                               # or translate at the boundary
```

- `Exception` **doesn’t** catch `KeyboardInterrupt` / `SystemExit` (those inherit from `BaseException`).
- Log details and re-raise (or convert to a clean error) at **application boundaries**.

### Always log (and usually re-raise)

```python
except Exception:
    logger.exception("Unhandled error in payment flow")
    raise
```

“Swallowing” hides problems. Logging tells you _what_ and _where_ it broke.

## 5) Anti-patterns → Better patterns

### Bare except + pass

```python
try:
    save()
except:
    pass
```

### Specific + clear handling

```python
try:
    save()
except PermissionError:
    notify_user("No write permission")
except OSError as e:
    logger.exception("Filesystem error")
    raise
```

## 6) FastAPI + Pydantic: how this applies

### Let Pydantic reject bad input (don’t catch it)

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr, conint

app = FastAPI()

class Signup(BaseModel):
    email: EmailStr
    age: conint(ge=13)

@app.post("/signup")
def signup(data: Signup):
    return {"ok": True}
```

If the request body is invalid, **FastAPI returns 422 automatically**.
You don’t need `try/except` here, don’t swallow validation errors.

### Translate **specific** failures to HTTP status codes

```python
from fastapi import HTTPException
from sqlalchemy.exc import IntegrityError
import httpx

@app.post("/users")
async def create_user(payload: dict):
    try:
        user = await repo.create_user(payload)  # may violate unique email
        return user
    except IntegrityError:
        raise HTTPException(409, "Email already exists")
    except httpx.TimeoutException:
        raise HTTPException(504, "Upstream timed out")
    except Exception as e:
        app.logger.exception("Unexpected error")
        raise HTTPException(500, "Internal server error")
```

**Point:** map known errors to the **right** status code; keep a final `except Exception` for safety.

### App-level last resort (still not bare)
