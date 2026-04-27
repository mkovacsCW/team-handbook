# Services

Services hold business logic that doesn't belong on a model and shouldn't sit inside a view. If a view is doing more than serializing input, calling something, and returning a response — that "something" probably belongs in a service.

## When to use a service

- Logic that touches **multiple models** (creating an order also writes an audit log and sends a notification).
- Logic reused across **multiple views or commands** (the same workflow runs from an API endpoint and a nightly cron).
- Logic that's **complex enough to test independently** of the request/response cycle.
- **External integrations** (calling a third-party API, generating a PDF, sending email).

If the logic is one model and one operation (`order.mark_paid()`), keep it as a model method instead.

## File layout

Start with a single `services.py` per app. Once functions get long or numerous, promote to a `services/` package and split files by concern:

```
orders/
├── services.py              # small app — one file is fine
└── ...

wip/
├── services/
│   ├── __init__.py
│   ├── answer_validation.py
│   ├── sheet_submission.py
│   └── reviewer_assignment.py
└── ...
```

Name files after the **concern**, not the model — `answer_validation.py` not `answer_services.py`. The "service" part is implied by the folder.

## Writing a service

Services are plain functions. No classes unless you genuinely need state.

```python
# orders/services.py
from django.db import transaction
from django.core.exceptions import ValidationError

from .models import Order, OrderItem
from notifications.services import notify_order_placed


@transaction.atomic
def create_order(*, company, worker, items):
    """Create an order and its line items, then notify the worker.

    Raises ValidationError if items is empty.
    Returns the created Order.
    """
    if not items:
        raise ValidationError("Order must have at least one item.")

    order = Order.objects.create(company=company, worker=worker)
    OrderItem.objects.bulk_create([
        OrderItem(order=order, **item) for item in items
    ])

    notify_order_placed(order)
    return order
```

Importing from a view:

```python
from orders.services import create_order

class OrderViewSet(viewsets.ModelViewSet):
    def perform_create(self, serializer):
        create_order(
            company=self.request.user.company,
            worker=self.request.user,
            items=serializer.validated_data['items'],
        )
```

We prefer `from orders.services import create_order` over `from orders import services` — call sites read more naturally and it's clearer at the import line which functions are being used.

## General best practices

- **Keyword-only arguments.** Use `*,` to force kwargs. Service signatures grow over time, and positional args are easy to mix up.
- **Wrap multi-write operations in `@transaction.atomic`.** If the second write fails, the first should roll back.
- **Raise exceptions on failure.** `ValidationError` for bad input, `PermissionDenied` for auth issues. Don't return `(success, result)` tuples — let exceptions bubble up to DRF.
- **Don't accept `request` objects.** Pass the specific things the service needs (`user`, `company`) so it's callable from a management command, a Celery task, or a test.
- **No HTTP concepts in services.** No `Response`, no status codes, no `request.data`. Those belong in views.
- **One public function per use case.** Helpers can be private (`_validate_items`).
- **Keep services importable in any direction with care.** Two services importing each other is a sign one of them is actually a helper for the other.

## Services vs. model methods vs. managers

| Logic involves…                              | Put it in…       |
| -------------------------------------------- | ---------------- |
| One model instance, simple state change      | Model method     |
| One model, querying/filtering                | Custom manager   |
| Multiple models, side effects, orchestration | Service function |
| External APIs, email, files, PDFs            | Service function |