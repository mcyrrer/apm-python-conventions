# Clean-code principles (with examples)

The reasoning and before/after snippets behind the always-on rules in
`clean-code.instructions.md`. When in doubt, optimize for the next reader.

## Single responsibility

If a function's name needs an "and", or you can extract another well-named
function from it, it's doing too much.

```python
# Bad — fetches AND renders
def fetch_and_show_users(): ...

# Good — one job each
def fetch_users() -> list[User]: ...
def render_users(users: list[User]) -> str: ...
```

## Limit arguments (≤ 3)

More arguments means more orderings to get wrong. Group related parameters into
a structured type.

```python
# Bad
def create_user(name, email, age, country, is_admin, newsletter): ...

# Good
@dataclass
class NewUser:
    name: str
    email: str
    age: int
    country: str

def create_user(user: NewUser) -> User: ...
```

## No flag parameters

A boolean that switches behaviour is two functions in disguise.

```python
# Bad
def notify(user, urgent=False): ...

# Good
def notify_urgent(user): ...
def notify_regular(user): ...
```

## No output arguments / side effects

Return values; don't mutate what you were handed or reach into globals.

```python
# Bad — mutates the caller's list
def add_defaults(items: list[str]) -> None:
    items.append("default")

# Good — returns a new value
def with_defaults(items: list[str]) -> list[str]:
    return [*items, "default"]
```

## Minimize nesting with early returns

```python
# Bad
def total(order):
    if order:
        if order.items:
            return sum(i.price for i in order.items)
    return 0

# Good
def total(order):
    if not order or not order.items:
        return 0
    return sum(i.price for i in order.items)
```

## Explanatory constants

```python
# Bad
if len(password) < 8: ...

# Good
MIN_PASSWORD_LENGTH = 8
if len(password) < MIN_PASSWORD_LENGTH: ...
```

## Specific exceptions

```python
# Bad
try:
    value = data[key]
except:
    value = None

# Good
try:
    value = data[key]
except KeyError:
    value = None
```

## Pythonic idioms

```python
# Bad
squares = []
for n in numbers:
    squares.append(n * n)

# Good
squares = [n * n for n in numbers]
```

Use generators for large sequences, context managers for resources, and
built-ins (`sum`, `any`, `enumerate`, `zip`) instead of manual equivalents.

## Comments & hygiene

- Docstrings on public modules/classes/functions; comments explain **why**, not
  **what**.
- Delete noise comments that restate the code, and never leave commented-out or
  dead code — version control is the history.
- Rewrite unclear code instead of annotating it with `# TODO: clean this up`.
