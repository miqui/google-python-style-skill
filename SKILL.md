# /python-style-guide

You are applying the **Google Python Style Guide** to a code review.
Check the code against every rule below and report violations using
the same tiered format as the main review (🔴 / 🟡 / 🟢). Cite the
exact line number or function name for each finding and show the fix.

---

## Imports

- `import x` for packages/modules only — never for individual classes or functions
- `from x import y` where x is package, y is module
- No relative imports — always use full package path
- One import per line (exception: `from typing import Any, Callable` is fine)
- No `import os, sys` on one line
- Four groups separated by blank lines, sorted lexicographically within each:
  1. `from __future__ import annotations`
  2. stdlib
  3. third-party
  4. internal / repo sub-packages

```python
# correct
import collections
import sys

import httpx
import structlog

from myproject.core import config
```

---

## Naming

| Type | Convention |
|---|---|
| Packages / modules | `lower_with_under` |
| Classes / exceptions / type aliases | `CapWords` |
| Functions / methods | `lower_with_under()` |
| Constants | `CAPS_WITH_UNDER` |
| Instance / global variables | `lower_with_under` |
| Private (module / class) | `_lower_with_under` |
| TypeVar unconstrained private | `_T`, `_P` |
| TypeVar constrained / bounded | `AddableType`, `AnyFunction` |

Rules:
- No single-char names except `i/j/k/v` counters, `e` in except, `f` in with
- No type embedded in name (`id_to_name_dict` → `id_to_name`)
- No dashes in package or module names — use underscores
- No `__dunder__` names (reserved by Python)

---

## Type Annotations

- Required on all public API functions and methods
- Do NOT annotate `self`, `cls`, or `__init__` return value
- Use `X | None` — NOT `Optional[X]`
- Never implicit optional: `a: str = None` is wrong; use `a: str | None = None`
- Use `Any` only when the type genuinely cannot be expressed
- Prefer abstract containers in signatures: `Sequence`, `Mapping` over `list`, `dict`
- Use built-in generics: `list[int]`, `dict[str, int]` — NOT `List[int]`, `Dict[str, int]`

```python
# correct
def fetch(keys: Sequence[str], limit: int | None = None) -> list[dict[str, Any]]: ...

# wrong
def fetch(keys: List[str], limit: Optional[int] = None) -> List[Dict]: ...
```

---

## Docstrings

- Always `"""triple double quotes"""`
- Summary line: one line, ≤80 chars, ends with `.` `?` or `!`
- Blank line after summary when body follows
- Required on: all public functions/methods/classes, any nontrivial function

Sections (in order when present):
- `Args:` — each param indented 4 spaces, wrapped at 80 chars
- `Returns:` (or `Yields:` for generators)
- `Raises:`
- `Attributes:` (class-level)

```python
def fetch_rows(
    table: Table,
    keys: Sequence[str],
    require_all: bool = False,
) -> Mapping[str, tuple[str, ...]]:
    """Fetches rows from a table.

    Args:
        table: An open Table instance.
        keys: Row keys to fetch.
        require_all: If True, only rows with all keys set are returned.

    Returns:
        A mapping of keys to row data tuples.

    Raises:
        IOError: An error occurred accessing the table.
    """
```

- `@property` docstrings: noun phrase — `"""The base URL."""` not `"""Returns the base URL."""`
- Generator docstrings: `Yields:` not `Returns:`
- Class docstring: describes what an *instance* represents, not "Class that..."

---

## Formatting

- **Line length**: 80 chars max
- **Indentation**: 4 spaces — never tabs
- **Semicolons**: prohibited
- **Backslash continuation**: prohibited — use implicit `()` joining
- **Trailing commas**: only when the closing bracket is on its own line
- **Blank lines**: 2 between top-level defs, 1 between methods

```python
# correct — implicit continuation
if (width == 0 and height == 0
        and color == "red"):
    pass

# wrong — backslash
if width == 0 \
        and height == 0:
    pass
```

Whitespace:
- No space inside `()`, `[]`, `{}`
- No space before `,` `;` `:`
- No spaces around `=` for keyword args WITHOUT annotation
- Space around `=` for keyword args WITH annotation: `def f(a: int = 0)`

String accumulation in loops — never `+=`:
```python
# correct
parts = []
for item in items:
    parts.append(str(item))
result = "".join(parts)

# wrong
result = ""
for item in items:
    result += str(item)
```

---

## Exceptions

- Names must end in `Error`; inherit from existing exceptions
- No bare `except:` — never catch `Exception` or `BaseException` unless re-raising
- Use `raise ValueError(...)` for violated preconditions — NOT `assert`
- Keep `try` blocks small; use `finally` for cleanup

```python
# correct
if minimum < 1024:
    raise ValueError(f"Minimum port must be >= 1024, got {minimum}.")

# wrong — assert is not guaranteed to run
assert minimum >= 1024
```

---

## Strings and Logging

- Use f-strings, `%`, or `.format()` for formatting — not `+` concatenation
- **Logging**: pass a pattern string as the first argument — NEVER an f-string

```python
# correct
logging.info("Processing user %s with role %s", user_id, role)

# wrong
logging.info(f"Processing user {user_id} with role {role}")
```

- Multi-line strings: use `textwrap.dedent` to avoid embedded leading spaces

---

## True/False Evaluations

```python
# correct — implicit falsiness for sequences/containers
if items:
if not items:

# wrong
if len(items) == 0:

# correct — always explicit for None
if foo is None:
if foo is not None:

# wrong
if not foo:          # can't distinguish False from None
if foo == None:
```

---

## Default Arguments

Never use mutable defaults — use `None` sentinel:

```python
# correct
def process(values: Sequence[int] | None = None) -> None:
    if values is None:
        values = []

def process(values: Sequence[int] = ()) -> None:  # immutable tuple OK

# wrong
def process(values: list[int] = []) -> None: ...     # mutable
def process(ts: float = time.time()) -> None: ...    # evaluated at import
```

---

## Comprehensions

- Simple single-clause list/dict/set comprehensions are fine
- Prohibited: multiple `for` clauses or nested filters

```python
# correct
result = [f(x) for x in items if predicate(x)]

# wrong — too complex
result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]
```

---

## Function Length

- No hard limit, but functions over ~40 lines should be flagged for potential split
- Each function should do one thing

---

## Main Guard

Always wrap top-level execution:

```python
def main() -> None:
    ...

if __name__ == "__main__":
    main()
```

Never execute meaningful code at module top level.

---

## Static vs Class Methods

- `@staticmethod`: avoid — write a module-level function instead
- `@classmethod`: only for named constructors or class-wide state management

---

## Files and Resources

Always use `with` statements — never rely on `__del__`:

```python
# correct
with open("data.txt") as f:
    content = f.read()
```

---

## Quick Reference

| Rule | Standard |
|---|---|
| Line length | 80 chars |
| Indentation | 4 spaces |
| Blank lines | 2 top-level, 1 between methods |
| Semicolons | Prohibited |
| Backslash continuation | Prohibited |
| Mutable defaults | Prohibited |
| Bare `except` | Prohibited |
| `assert` for logic | Prohibited |
| `Optional[X]` | Use `X \| None` |
| `List[X]` / `Dict` | Use `list[X]` / `dict` |
| Logging f-strings | Use pattern strings |
| `@staticmethod` | Use module-level function |
| String `+=` in loops | Use list + `"".join()` |
