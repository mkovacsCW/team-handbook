# Utils

Utils are small, **stateless**, **reusable** helpers. The test for "is this a util?" — could it be lifted into another project unchanged? If yes, it's a util. If it knows about your business rules, it's a service.

## Utils vs. services

| It's a util if…                       | It's a service if…                       |
| ------------------------------------- | ---------------------------------------- |
| Pure function, no DB writes           | Touches the DB or external systems       |
| No knowledge of your domain           | Encodes business rules                   |
| `format_phone_number(value)`          | `create_order(...)`                      |
| `parse_excel_row(row)`                | `import_orders_from_excel(file)`         |
| `generate_random_token(length=32)`    | `issue_password_reset_token(user)`       |

A handy way to think about it: utils answer "how do I do this thing?", services answer "what should happen when this thing occurs?".

## File layout

Same pattern as services. Start with `utils.py` per app, promote to a `utils/` package once it grows:

```
orders/
├── utils.py                 # small — one file is fine
└── ...

wip/
├── utils/
│   ├── __init__.py
│   ├── excel.py
│   ├── dates.py
│   └── phone.py
└── ...
```

Name files after the **kind of thing they help with** — `excel.py`, `dates.py` — not after the app or model.

## Writing a util

```python
# wip/utils/dates.py
from datetime import date, timedelta


def previous_business_day(d: date) -> date:
    """Return the most recent weekday strictly before ``d``."""
    d = d - timedelta(days=1)
    while d.weekday() >= 5:  # 5=Sat, 6=Sun
        d -= timedelta(days=1)
    return d


def week_range(d: date) -> tuple[date, date]:
    """Return (Monday, Sunday) of the week containing ``d``."""
    monday = d - timedelta(days=d.weekday())
    return monday, monday + timedelta(days=6)
```

Usage:

```python
from wip.utils.dates import previous_business_day

cutoff = previous_business_day(date.today())
```

## Project-wide vs. app-local

If a util is **only** used by one app, it stays in that app. The moment a second app needs it, move it to a project-level location (e.g. a `core/utils/` app) and update imports — don't cross-import from a sibling app's `utils/`.

## General best practices

- **Pure functions.** Same input → same output. No surprise DB queries, no hidden global state.
- **Type hints on signatures.** Utils get reused in unfamiliar contexts; types document intent.
- **No Django models in utility code where possible.** A util that takes a `User` instance is fine; a util that calls `User.objects.filter(...)` should probably be a service or a manager method.
- **Docstrings with examples for non-obvious helpers.** `previous_business_day(date(2024, 1, 1))` — what does it return? A docstring example saves a future reader from running it in a shell.
- **No I/O in deep utils.** A function that *reads* a file is a util. A function that *fetches a URL and parses the response and writes a record* is a service that uses utils.
- **Don't dump everything into `utils.py`.** If your `utils.py` is over ~300 lines or covers three unrelated concerns, split it.