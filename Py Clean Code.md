# Python Clean Code 


## What is Clean Code?

Clean code refers to **well-written, easy-to-understand, and maintainable code**. It’s not just about making the program run—it’s about writing it in a way that other developers (and your future self) can easily read, understand, and extend.  

### Key traits of clean code:
- **Readable**: Anyone can understand what the code does without guessing.  
- **Simple**: Avoids unnecessary complexity.  
- **Consistent**: Follows naming conventions, formatting, and style guides.  
- **Modular**: Broken into small, reusable functions or classes.  
- **Well-documented**: Explains *why* something is done, not just *what*.  

---

## Why is Clean Code Needed?

Writing clean code is essential because software is rarely written once and forgotten—it evolves. Clean code makes this evolution easier.  

- **Collaboration**: Teams can work together without confusion.  
- **Maintenance**: Bugs are easier to find and fix.  
- **Scalability**: Adding new features doesn’t break old ones.  
- **Longevity**: Code remains useful and understandable years later.  
- **Efficiency**: Saves time and money in the long run.  

---




##  We Read Code More Than We Write
- **Maintenance dominates development**: After initial creation, most of a software project’s life is spent fixing bugs, adding features, or adapting to new requirements.  
- **Team collaboration**: Developers constantly read each other’s code to understand how things work before making changes.  
- **Debugging and reviews**: Code reviews, testing, and troubleshooting require careful reading.  
- **Learning and onboarding**: New team members spend significant time reading existing code to get up to speed.  

---

## Ratio of Reading vs Writing
Studies and industry experience suggest that developers spend **70–90% of their time reading code** and only **10–30% writing new code**.  
- Writing is quick — but understanding what’s already there takes much longer.  
- If code is messy, the reading time skyrockets.  

---
##  What Happens Without Clean Code?
If code is messy, unclear, or inconsistent, problems pile up quickly:  

- **Hard to read**: Developers waste time figuring out what the code does.  
- **Bug-prone**: Errors hide in tangled logic.  
- **Expensive to maintain**: Fixing or updating takes much longer.  
- **Team frustration**: Collaboration breaks down when no one understands the code.  
- **Technical debt**: Shortcuts today create huge problems tomorrow.  

Think of it like building a house: if the wiring and plumbing are messy, the house may still stand—but fixing or upgrading it later becomes a nightmare.  

---
## Why This Matters
Because reading dominates, **code should be optimized for readability, not just execution**.  
- Clean code reduces cognitive load.  
- It makes collaboration smoother.  
- It prevents errors caused by misunderstandings.  

---

## Complicated Code Equal to Bad Code?
Not always, but **most of the time complicated code is bad code**. Let’s break it down:

### When Complicated Code = Bad Code
- **Unnecessary complexity**: If the code could be written in a simpler way but isn’t, that’s bad.  
- **Hard to read**: If developers struggle to understand what’s happening, it slows down progress.  
- **Bug-prone**: Complex logic often hides errors.  
- **Poor maintainability**: Future changes become risky and expensive.  

This is what people usually mean when they say *complicated code is bad code*.  

---

###  When Complicated Code ≠ Bad Code
Sometimes complexity is **unavoidable**:
- **Complex problems**: Algorithms for cryptography, machine learning, or operating systems are inherently complex.  
- **Performance optimization**: Code may look complicated because it’s tuned for speed or memory efficiency.  
- **Domain-specific logic**: Business rules or scientific calculations can be intricate by nature.  

In these cases, complexity isn’t “bad” — but it must be handled with **clean coding practices** (good naming, modularization, documentation) so that others can still understand it.  

## Example: Factorial Function

### Complicated (Bad) Code
```python
def f(x):
    if x == 0:
        return 1
    else:
        y = 1
        for i in range(1, x+1):
            for j in range(1, 2):  # unnecessary loop
                y = y * i
        return y
```

###  Clean (Good) Code
```python
def factorial(n: int) -> int:
    """Return the factorial of n using iteration."""
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result
```

---

##  Detailed Comparison

| Aspect              | Complicated (Bad) Code | Clean (Good) Code |
|---------------------|------------------------|------------------|
| **Naming**          | Function named `f` (unclear) | Function named `factorial` (descriptive) |
| **Readability**     | Nested unnecessary loop makes logic harder to follow | Straightforward loop, easy to read |
| **Documentation**   | No explanation | Docstring explains purpose |
| **Maintainability** | Harder to modify or debug | Easy to extend or change |
| **Efficiency**      | Extra loop adds wasted computation | Efficient, only necessary operations |
| **Clarity**         | Reader must “decode” what’s happening | Reader immediately understands intent |

---


**Refining principles and examples**

The user wants a detailed but simple response, focused only on ASCII and Python code. I’ll provide structured explanations with examples, including well-known principles like DRY, error handling, and logging. For formatting, I won’t use bold headings or markdown—just plain ASCII characters. I can still use hyphens and clear labels for each principle, and I’ll make sure to include Python examples with triple backticks. Whoever requires simplicity means avoiding any extras, so I’ll keep it clean and plain but with enough detail to guide any coding style questions. Let’s proceed!
HOW TO WRITE CLEAN PYTHON CODE
- Clear, readable code saves time for you and your teammates.
- Aim for simplicity, consistency, and explicit intent.
- Prefer small, focused functions and meaningful names.
- Document why, type your interfaces, and test behavior.

PRINCIPLES WITH EXAMPLES

1) USE MEANINGFUL NAMES
- Names should tell the reader what the thing is or does.

```python
# Bad
def fn(a, b):
    return a * b + 10

# Good
def compute_score(weight: float, value: float) -> float:
    return weight * value + 10.0
```

2) WRITE SMALL, FOCUSED FUNCTIONS
- Each function should do one thing and do it well.

```python
# Bad: does parsing, validation, and computation in one place
def process(line: str) -> int:
    parts = line.split(",")
    if len(parts) != 2:
        raise ValueError("invalid")
    x = int(parts[0])
    y = int(parts[1])
    return (x * y) // 2

# Good: single-responsibility functions
def parse_pair(line: str) -> tuple[int, int]:
    parts = line.split(",")
    if len(parts) != 2:
        raise ValueError("expected two comma-separated values")
    return int(parts[0]), int(parts[1])

def compute_area(x: int, y: int) -> int:
    return (x * y) // 2

def process(line: str) -> int:
    x, y = parse_pair(line)
    return compute_area(x, y)
```

3) BE EXPLICIT AND CONSISTENT
- Avoid magic numbers, prefer constants and clear defaults.

```python
# Bad
def connect(timeout=7):
    ...

# Good
DEFAULT_TIMEOUT_SECONDS = 7

def connect(timeout: int = DEFAULT_TIMEOUT_SECONDS) -> None:
    ...
```

4) USE DOCSTRINGS TO EXPLAIN WHY (NOT JUST WHAT)
- Keep docstrings short, state behavior, parameters, returns, and any invariants.

```python
def percentile(values: list[float], p: float) -> float:
    """
    Return the p-th percentile (0 to 100) for a non-empty list of values.
    Uses linear interpolation between closest ranks.
    Raises ValueError if values is empty or p is out of range.
    """
    if not values:
        raise ValueError("values must be non-empty")
    if not (0.0 <= p <= 100.0):
        raise ValueError("p must be between 0 and 100")
    values = sorted(values)
    k = (len(values) - 1) * (p / 100.0)
    f = int(k)
    c = min(f + 1, len(values) - 1)
    if f == c:
        return values[int(k)]
    return values[f] + (values[c] - values[f]) * (k - f)
```

5) TYPE HINTS CLARIFY INTERFACES
- Type hints improve readability and catch errors early.

```python
from typing import Iterable, Iterator

def normalize(xs: Iterable[float]) -> list[float]:
    total = sum(xs)
    if total == 0:
        return [0.0 for _ in xs]
    return [x / total for x in xs]

def batched(xs: Iterable[int], size: int) -> Iterator[list[int]]:
    """
    Yield lists of length `size` (last batch may be shorter).
    """
    batch: list[int] = []
    for x in xs:
        batch.append(x)
        if len(batch) == size:
            yield batch
            batch = []
    if batch:
        yield batch
```

6) GUARD CLAUSES AND EARLY RETURNS
- Reduce nesting by exiting as soon as conditions fail.

```python
# Bad
def discount(price: float, code: str | None) -> float:
    if code is not None:
        if price > 0:
            if code == "PROMO10":
                return price * 0.9
            else:
                return price
        else:
            return 0.0
    else:
        return price

# Good
def discount(price: float, code: str | None) -> float:
    if price <= 0:
        return 0.0
    if code != "PROMO10":
        return price
    return price * 0.9
```

7) AVOID DUPLICATION (DRY)
- Extract common logic; duplication multiplies bugs.

```python
# Bad
def render_user_html(user):
    return "<div>" + user.name + "</div>"

def render_admin_html(admin):
    return "<div>" + admin.name + "</div>"

# Good
def render_box(name: str) -> str:
    return f"<div>{name}</div>"

def render_user_html(user) -> str:
    return render_box(user.name)

def render_admin_html(admin) -> str:
    return render_box(admin.name)
```

8) HANDLE ERRORS DELIBERATELY
- Validate inputs, catch only specific exceptions, provide helpful messages.

```python
# Bad
def read_json(path: str):
    import json
    return json.load(open(path))

# Good
import json
from pathlib import Path

def read_json(path: str) -> dict:
    p = Path(path)
    if not p.exists():
        raise FileNotFoundError(f"no such file: {p}")
    with p.open("r", encoding="utf-8") as f:
        try:
            return json.load(f)
        except json.JSONDecodeError as e:
            raise ValueError(f"invalid JSON in {p}: {e}") from e
```

9) PREFER COMPOSITION OVER SIDE EFFECTS
- Keep functions pure when possible; isolate I/O.

```python
# Pure computation
def total_with_tax(subtotal: float, rate: float) -> float:
    return subtotal * (1.0 + rate)

# I/O separated
def print_invoice(subtotal: float, rate: float) -> None:
    total = total_with_tax(subtotal, rate)
    print(f"Total: {total:.2f}")
```

10) USE CONTEXT MANAGERS
- They make resource handling safe and clear.

```python
# Bad
f = open("data.txt", "r")
data = f.read()
f.close()

# Good
from pathlib import Path

def read_text(path: str) -> str:
    p = Path(path)
    with p.open("r", encoding="utf-8") as f:
        return f.read()
```

11) READABLE LOOPS AND COMPREHENSIONS
- Use comprehensions for simple transforms; avoid cramming complex logic.

```python
# Good simple transform
squared = [x * x for x in range(10) if x % 2 == 0]

# Too clever (avoid)
squared = [x**2 if x % 2 == 0 else None for x in range(10) if x % 2 == 0]
```

12) FORMAT CONSISTENTLY
- Follow PEP 8: spaces, line length, imports, naming.

```python
# Bad spacing and mixed styles
import os,sys
def add(a,b):return a+b

# Good
import os
import sys

def add(a: int, b: int) -> int:
    return a + b
```

13) ISOLATE CONFIGURATION AND CONSTANTS
- Centralize values that may change.

```python
API_URL = "https://api.example.com"
RETRY_LIMIT = 3

def fetch_data(session, endpoint: str) -> dict:
    url = f"{API_URL}/{endpoint}"
    for attempt in range(RETRY_LIMIT):
        resp = session.get(url, timeout=10)
        if resp.ok:
            return resp.json()
    raise TimeoutError(f"failed to fetch {url} after {RETRY_LIMIT} retries")
```

14) DESIGN CLEAR DATA STRUCTURES
- Prefer dataclasses for structured data.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float

def distance(a: Point, b: Point) -> float:
    dx = a.x - b.x
    dy = a.y - b.y
    return (dx * dx + dy * dy) ** 0.5
```

15) SEPARATE LAYERS (BOUNDARIES)
- Keep domain logic separate from frameworks and I/O.

```python
# Domain
def calculate_final_price(base: float, tax_rate: float, discount_rate: float) -> float:
    taxed = base * (1 + tax_rate)
    discounted = taxed * (1 - discount_rate)
    return round(discounted, 2)

# Application boundary
def checkout(cart_total: float) -> None:
    final = calculate_final_price(cart_total, tax_rate=0.18, discount_rate=0.10)
    print(f"Final: {final}")
```

16) WRITE TESTABLE CODE
- Pure functions and clear interfaces are easier to test.

```python
def slugify(name: str) -> str:
    """
    Lowercase, replace spaces with hyphens, strip non-alphanumerics.
    """
    import re
    s = name.lower().strip()
    s = re.sub(r"\s+", "-", s)
    s = re.sub(r"[^a-z0-9-]", "", s)
    return s

def test_slugify():
    assert slugify("Hello World!") == "hello-world"
    assert slugify("  A  B  ") == "a-b"
```

17) LOG INTENTIONALLY
- Log at the right level; make messages actionable.

```python
import logging

log = logging.getLogger(__name__)

def run_job(job_id: str) -> None:
    log.info("starting job %s", job_id)
    try:
        # ... work ...
        log.debug("job %s progress: step=%s", job_id, "prepare")
        # ...
    except Exception:
        log.exception("job %s failed", job_id)
        raise
    else:
        log.info("job %s completed", job_id)
```

18) AVOID PREMATURE OPTIMIZATION
- Write clear code first, optimize hotspots with profiling.

```python
# Clear baseline
def unique_sorted(xs: list[int]) -> list[int]:
    return sorted(set(xs))

# Micro-optimized version should be justified by measurement
```

19) USE ITERATORS AND GENERATORS FOR LARGE DATA
- Stream data to reduce memory usage.

```python
def read_lines(path: str):
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            yield line.rstrip("\n")

def count_words(path: str) -> int:
    return sum(len(line.split()) for line in read_lines(path))
```

20) PROVIDE SAFE DEFAULTS
- Fail fast on bad input; offer predictable behavior.

```python
def paginate(items: list[int], page: int, size: int) -> list[int]:
    if page < 1 or size < 1:
        raise ValueError("page and size must be positive")
    start = (page - 1) * size
    end = start + size
    return items[start:end]
```

21) MAKE SIDE EFFECTS OBVIOUS
- Name functions with verbs when they perform I/O or mutations.

```python
def load_config(path: str) -> dict:
    ...

def update_user_email(user_id: str, email: str) -> None:
    ...
```

22) DOCUMENT ASSUMPTIONS AND EDGE CASES
- Explain constraints that are not obvious from code.

```python
def divide(a: float, b: float) -> float:
    """
    Divide a by b.
    Assumes b != 0; callers must validate b or catch ZeroDivisionError.
    """
    return a / b
```

23) KEEP PUBLIC APIS SMALL
- Hide internals, expose minimal necessary functions.

```python
# _helper is private by convention
def _normalize_whitespace(s: str) -> str:
    return " ".join(s.split())

def clean_title(title: str) -> str:
    return _normalize_whitespace(title).title()
```

24) CONSISTENT RETURN TYPES
- Avoid mixing None and values unless documented.

```python
# Bad
def find_user(id: str):
    # sometimes returns dict, sometimes None, sometimes raises
    ...

# Good
from typing import Optional

def find_user(id: str) -> Optional[dict]:
    """
    Return user dict if found, else None.
    Does not raise on missing user.
    """
    ...
```

25) WRITE SIMPLE, DIRECT CONTROL FLOW
- Prefer straightforward loops and conditionals over clever tricks.

```python
# Bad: clever but obscure
result = condition and x or y

# Good
if condition:
    result = x
else:
    result = y
```

PUTTING IT ALL TOGETHER (A SMALL, CLEAN EXAMPLE)

```python
from dataclasses import dataclass
from typing import Optional

DEFAULT_TAX_RATE = 0.18
DEFAULT_DISCOUNT_RATE = 0.10

@dataclass(frozen=True)
class Item:
    name: str
    price: float

@dataclass
class Cart:
    items: list[Item]

    def subtotal(self) -> float:
        return sum(item.price for item in self.items)

def calculate_total(subtotal: float, tax_rate: float, discount_rate: float) -> float:
    """
    Calculate final total with tax and discount.
    Raises ValueError for invalid rates.
    """
    if not (0.0 <= tax_rate <= 1.0):
        raise ValueError("tax_rate must be between 0 and 1")
    if not (0.0 <= discount_rate <= 1.0):
        raise ValueError("discount_rate must be between 0 and 1")
    taxed = subtotal * (1 + tax_rate)
    discounted = taxed * (1 - discount_rate)
    return round(discounted, 2)

def summarize(cart: Cart, tax_rate: float = DEFAULT_TAX_RATE, discount_rate: float = DEFAULT_DISCOUNT_RATE) -> str:
    """
    Return a human-readable summary for the cart.
    """
    sub = cart.subtotal()
    total = calculate_total(sub, tax_rate, discount_rate)
    lines = [
        f"Items: {len(cart.items)}",
        f"Subtotal: {sub:.2f}",
        f"Tax rate: {tax_rate:.2f}",
        f"Discount rate: {discount_rate:.2f}",
        f"Total: {total:.2f}",
    ]
    return "\n".join(lines)

def main() -> None:
    cart = Cart(items=[Item("Pen", 20.0), Item("Notebook", 80.0)])
    print(summarize(cart))

if __name__ == "__main__":
    main()
```

CHECKLIST (QUICK REFERENCE)
- Meaningful names
- Small, focused functions
- Docstrings that explain behavior and constraints
- Type hints on public interfaces
- Guard clauses and early returns
- No duplication
- Explicit error handling
- Pure functions for core logic; isolate I/O
- Context managers for resources
- Simple comprehensions; avoid cleverness
- Consistent formatting (PEP 8)
- Centralized configuration/constants
- Dataclasses for structured data
- Clear boundaries between layers
- Tests for important behavior
- Intentional logging
- Optimize only after measuring
- Iterators/generators for large data
- Safe defaults and predictable returns
- Minimal public surface area


---
## Writing Pythonic Code

WHAT IS PYTHONIC
- "Pythonic" means writing code that follows Python's philosophy and idioms.
- It emphasizes readability, simplicity, and elegance.
- Pythonic code uses built-in features and conventions naturally.
- The Zen of Python (`import this`) summarizes the mindset:
  - Simple is better than complex.
  - Readability counts.
  - There should be one obvious way to do it.

HOW TO WRITE PYTHONIC CODE
- Prefer readability over clever tricks.
- Use built-in functions and idioms.
- Follow PEP 8 style guide.
- Write small, clear functions.
- Handle errors with exceptions (EAFP).
- Use comprehensions, enumerate, zip, and context managers.

EXAMPLES

1) READABLE AND SIMPLE
```python
# Not Pythonic
def square_list(nums):
    result = []
    for i in range(len(nums)):
        result.append(nums[i] * nums[i])
    return result

# Pythonic
def square_list(nums):
    return [n * n for n in nums]
```

2) BUILT-IN FUNCTIONS
```python
# Not Pythonic
total = 0
for n in [1, 2, 3]:
    total += n

# Pythonic
total = sum([1, 2, 3])
```

3) ENUMERATE AND ZIP
```python
# Not Pythonic
i = 0
for item in ["a", "b", "c"]:
    print(i, item)
    i += 1

# Pythonic
for i, item in enumerate(["a", "b", "c"]):
    print(i, item)

names = ["Alice", "Bob"]
scores = [85, 90]
for name, score in zip(names, scores):
    print(name, score)
```

4) CONTEXT MANAGERS
```python
# Not Pythonic
f = open("data.txt")
content = f.read()
f.close()

# Pythonic
with open("data.txt") as f:
    content = f.read()
```

5) EAFP (Easier to Ask Forgiveness than Permission)
```python
# Not Pythonic
if "key" in my_dict:
    value = my_dict["key"]
else:
    value = None

# Pythonic
try:
    value = my_dict["key"]
except KeyError:
    value = None
```

6) PEP 8 STYLE
```python
# Not Pythonic
def add(a,b):return a+b

# Pythonic
def add(a: int, b: int) -> int:
    return a + b
```

SUMMARY CHECKLIST
- Use meaningful names
- Prefer list/dict comprehensions
- Use built-ins like sum, max, any, all
- Apply enumerate and zip for iteration
- Manage resources with `with`
- Handle errors with exceptions
- Follow PEP 8 formatting
- Keep code simple and readable

---

KISS, DRY, NIH PRINCIPLES IN SOFTWARE DEVELOPMENT

KISS (Keep It Simple, Stupid)
- Principle: Simplicity is better than complexity.
- Code should be easy to read, understand, and maintain.
- Avoid over-engineering.

Example:
```python
# Not KISS: overly complex
def add_numbers(a: int, b: int) -> int:
    result = 0
    for i in [a, b]:
        result += i
    return result

# KISS: simple and clear
def add_numbers(a: int, b: int) -> int:
    return a + b
```

---

DRY (Don't Repeat Yourself)
- Principle: Avoid duplication of code.
- Extract common logic into reusable functions or modules.
- Duplication increases maintenance cost and risk of bugs.

Example:
```python
# Not DRY: duplicated logic
def area_rectangle(w: float, h: float) -> float:
    return w * h

def area_square(s: float) -> float:
    return s * s  # duplication of multiplication logic

# DRY: reuse logic
def area_rectangle(w: float, h: float) -> float:
    return w * h

def area_square(s: float) -> float:
    return area_rectangle(s, s)
```

---

NIH (Not Invented Here)
- Principle: Avoid reinventing solutions that already exist.
- Use standard libraries and frameworks instead of writing your own.
- Reinventing wastes time and may introduce errors.

Example:
```python
# NIH violation: reinventing string reversal
def reverse_string_manual(s: str) -> str:
    result = ""
    for i in range(len(s) - 1, -1, -1):
        result += s[i]
    return result

# Correct: use Python built-in features
def reverse_string(s: str) -> str:
    return s[::-1]
```

---

SUMMARY
- KISS: Keep code simple and clear.
- DRY: Avoid duplication, reuse logic.
- NIH: Use existing libraries and solutions instead of reinventing.

---

CONTEXT MANAGER IN PYTHON

DEFINITION
- A context manager is a construct that properly manages resources.
- It defines setup and teardown logic using `__enter__` and `__exit__` methods.
- Commonly used with the `with` statement.
- Ensures resources are acquired and released safely (files, network connections, locks).

BENEFITS
- Automatic cleanup of resources.
- Reduces boilerplate code.
- Prevents resource leaks.
- Improves readability and reliability.

BUILT-IN EXAMPLE (FILE HANDLING)
```python
# Without context manager (manual cleanup)
f = open("data.txt", "r")
try:
    content = f.read()
finally:
    f.close()

# With context manager (automatic cleanup)
with open("data.txt", "r") as f:
    content = f.read()
```

CUSTOM CONTEXT MANAGER USING CLASS
```python
class ManagedFile:
    def __init__(self, filename):
        self.filename = filename

    def __enter__(self):
        self.file = open(self.filename, "w")
        return self.file

    def __exit__(self, exc_type, exc_value, traceback):
        if self.file:
            self.file.close()

# Usage
with ManagedFile("example.txt") as f:
    f.write("Hello, Context Manager!")
```

CUSTOM CONTEXT MANAGER USING DECORATOR
```python
from contextlib import contextmanager

@contextmanager
def managed_file(name):
    f = open(name, "w")
    try:
        yield f
    finally:
        f.close()

# Usage
with managed_file("example.txt") as f:
    f.write("Hello via contextlib!")
```

EXAMPLE WITH ERROR HANDLING
```python
class Demo:
    def __enter__(self):
        print("Entering context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Exiting context")
        if exc_type:
            print(f"Error: {exc_value}")
        return True  # suppress exception

# Usage
with Demo() as d:
    raise ValueError("Something went wrong")
```

---

## PEP (Python Enhancement Proposal)

In Python, a PEP (Python Enhancement Proposal) is a formal document that describes new features, design changes, or processes for the Python language.** It serves as the primary mechanism for proposing, discussing, and standardizing improvements in Python, with each PEP assigned a unique number and maintained as part of the official record. 

---
## Test-Driven Development (TDD)

## What is Test-Driven Development (TDD)?
**Test-Driven Development (TDD)** is a software development methodology where you write **tests before writing the actual code**. The idea is to ensure that your code meets the requirements from the very beginning and remains reliable as it evolves.

---

## The TDD Cycle (Red → Green → Refactor)
1. **Write a Test (Red)**  
   - Write a unit test that describes the desired functionality.  
   - At this point, the test will fail because the feature isn’t implemented yet.

2. **Write the Code (Green)**  
   - Implement the minimum code necessary to make the test pass.  
   - Run the test again, and if it passes, you know the code works for that case.

3. **Refactor**  
   - Clean up the code while keeping all tests passing.  
   - This ensures maintainability without breaking functionality.

---

## TDD in Python
Python makes TDD easy thanks to its built-in **`unittest`** module and third-party tools like **pytest**.

### Example:
```python
# test_calculator.py
import unittest
from calculator import add

class TestCalculator(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)

if __name__ == "__main__":
    unittest.main()
```

Then you write the actual code:

```python
# calculator.py
def add(a, b):
    return a + b
```

- First, the test fails (because `add` doesn’t exist yet).  
- After writing `add`, the test passes.  
- You can then refactor if needed, while tests ensure nothing breaks.

---

## Benefits of TDD
- **Reliability**: Bugs are caught early.  
- **Confidence**: You can refactor code without fear of breaking functionality.  
- **Documentation**: Tests serve as living documentation of how the code should behave.  
- **Design**: Encourages writing modular, clean, and testable code.


---
## Naming in Python


##  General Rules for Naming in Python
- Names must follow **identifier rules**:
  - Only letters (a–z, A–Z), digits (0–9), and underscore (`_`) are allowed.
  - Names **cannot start with a digit**.
  - Names are **case-sensitive** (`Var` and `var` are different).
  - Reserved keywords (like `class`, `def`, `return`) cannot be used as names.

---

## Variables
- **Style**: Use **snake_case** → all lowercase with underscores between words.
  - Example: `user_name`, `total_amount`, `is_valid`
- **Avoid**:
  - Starting with numbers →  `1value`
  - Using special characters →  `value$`, `name@`
  - Single letters unless in short loops → `i`, `j` are fine in loops, but not for meaningful data.
  - Overly long names →  `this_is_a_variable_that_stores_the_total_amount_of_items_in_cart`
- **Good practice**:
  - Be descriptive but concise →  `cart_total`

---

##  Functions
- **Style**: Also use **snake_case**.
  - Example: `calculate_area()`, `send_email()`
- **Avoid**:
  - Capital letters →  `CalculateArea()`
  - Ambiguous names →  `do_stuff()`
- **Good practice**:
  - Use verbs that describe the action →  `get_user_data()`, `update_record()`

---

##  Classes
- **Style**: Use **PascalCase** (a.k.a. CamelCase with first letter capitalized).
  - Example: `UserProfile`, `BankAccount`, `CarModel`
- **Avoid**:
  - snake_case →  `user_profile`
  - Starting with lowercase →  `bankaccount`
- **Good practice**:
  - Class names should be nouns representing entities →  `Invoice`, `Student`

---

##  Constants
- **Style**: Use **UPPERCASE_WITH_UNDERSCORES**.
  - Example: `PI = 3.14159`, `MAX_USERS = 100`
- **Avoid**:
  - Lowercase →  `pi = 3.14159`
  - Mixed styles →  `PiValue`

---

##  Underscore Usage
- Leading underscore (`_var`) → indicates **internal use** (weak "private").
- Double leading underscore (`__var`) → triggers **name mangling** in classes.
- Double underscores around (`__init__`) → reserved for **special methods** (dunder methods).
- Avoid unnecessary underscores at the end →  `var__`


---

## Things to Avoid
- Names starting with numbers → `123abc`
- Using spaces → `user name`
- Using special characters → `user-name`, `value@home`
- Overusing underscores → `___var___` (unless dunder methods)
- Shadowing built-in names →  `list = [1,2,3]`, `str = "hello"`

---

In short:  
- **Variables & functions** → `snake_case`  
- **Classes** → `PascalCase`  
- **Constants** → `UPPERCASE`  
- **Never start with numbers, avoid special chars, don’t shadow built-ins.**

---

## Summary: Writing Clean Code

Clean code refers to writing software that is simple, readable, and easy to maintain. It emphasizes clarity through meaningful names, consistent formatting, and straightforward structure, while avoiding unnecessary complexity. Developers are encouraged to break large functions into smaller, single-purpose ones, use comments only when essential, and follow principles like DRY (Don’t Repeat Yourself) and SOLID to ensure good design. Regular refactoring and thorough testing help keep code reliable and adaptable. By following these practices, clean code reduces technical debt, improves collaboration, speeds up debugging, and ultimately leads to higher-quality software that is easier to extend and more satisfying for users.