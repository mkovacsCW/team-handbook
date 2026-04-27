# Exceptions & Error Handling

Two things to keep straight: **what to raise** in business logic, and **how it gets turned into an HTTP response** at the API boundary.

## Which exception to raise

Use Django/DRF's built-in exceptions where they fit. They map to sensible HTTP responses automatically when raised inside a DRF view.

| Situation                                    | Raise                                  | HTTP |
| -------------------------------------------- | -------------------------------------- | ---- |
| Invalid input from the client                | `rest_framework.exceptions.ValidationError` | 400  |
| User is not authenticated                    | `NotAuthenticated`                     | 401  |
| User is authenticated but can't do this      | `PermissionDenied`                     | 403  |
| Resource doesn't exist                       | `Http404` (or `get_object_or_404`)     | 404  |
| Conflict (duplicate, already processed)      | Custom exception, see below            | 409  |
| Something on our side broke                  | Let it propagate — 500                 | 500  |

`django.core.exceptions.ValidationError` and `rest_framework.exceptions.ValidationError` are **different classes**. In services called from DRF views, prefer the DRF one — it serializes to the response cleanly. The Django one only auto-converts in some places.

## Raising from a service

Services raise; views don't catch (mostly). DRF turns the exception into the right response.

```python
# orders/services.py
from rest_framework.exceptions import ValidationError, PermissionDenied
from django.shortcuts import get_object_or_404

from .models import Order


def cancel_order(*, order_id, user):
    order = get_object_or_404(Order, pk=order_id)

    if order.company != user.company:
        raise PermissionDenied("You can't cancel orders from another company.")

    if order.status == 'shipped':
        raise ValidationError("Shipped orders can't be cancelled.")

    order.status = 'cancelled'
    order.save(update_fields=['status'])
    return order
```

The view stays thin:

```python
class OrderViewSet(viewsets.ModelViewSet):
    @action(detail=True, methods=['post'])
    def cancel(self, request, pk=None):
        order = cancel_order(order_id=pk, user=request.user)
        return Response(OrderSerializer(order).data)
```

No `try/except`. If `cancel_order` raises `ValidationError`, DRF returns a 400 with the message. If it raises `PermissionDenied`, DRF returns 403.

## Custom exceptions

When the built-ins don't capture the meaning, define your own. Subclass `APIException` so DRF still handles the response automatically:

```python
# orders/exceptions.py
from rest_framework.exceptions import APIException


class OrderAlreadyProcessed(APIException):
    status_code = 409
    default_detail = "This order has already been processed."
    default_code = "order_already_processed"


class InventoryUnavailable(APIException):
    status_code = 409
    default_detail = "One or more items are out of stock."
    default_code = "inventory_unavailable"
```

Raise them like any other exception:

```python
if order.is_processed:
    raise OrderAlreadyProcessed()
```

Custom exceptions are worth it when:

- The same condition gets raised from multiple places and you want one source of truth.
- The frontend needs to distinguish this error from other 400s — the `default_code` lets the client switch on it.
- The message is non-obvious and benefits from being defined once.

## When to actually catch

Catching exceptions is the exception, not the rule. Do catch when:

- **Calling external APIs** that can fail transiently — wrap in `try/except` and decide whether to retry, log, or re-raise as a domain exception.
- **Inside a batch/cron loop** — one bad row shouldn't kill 999 good ones. Catch, log, count, continue.
- **Translating a low-level error** into a domain one (`IntegrityError` → `OrderAlreadyProcessed`).

```python
# Good: per-record catching in a batch
for row in rows:
    try:
        import_row(row)
        result['imported'] += 1
    except ValidationError as e:
        logger.warning("Skipped row %s: %s", row.id, e)
        result['skipped'] += 1
    except Exception:
        logger.exception("Failed row %s", row.id)
        result['errors'] += 1
```

What you should **not** do is catch broadly and swallow. `except Exception: pass` hides real bugs.

## General best practices

- **Raise specific exceptions, not generic ones.** `ValidationError("Email already in use")` not `Exception("bad")`.
- **Don't use exceptions for control flow.** If "not found" is an expected outcome of a query, return `None`. If it means the request is invalid, raise.
- **Don't catch what you can't handle.** Let DRF deal with it. The default exception handler is good.
- **Use `logger.exception(...)` inside `except`.** It captures the traceback. `logger.error(str(e))` loses it.
- **`get_object_or_404` over `try/except DoesNotExist`** at the boundary. Cleaner.
- **Validation belongs in serializers when it's about shape**, in services when it's about business rules. "Email is required" → serializer. "This user can't be promoted to admin without 2FA enabled" → service.

## Quick reference

| You want to…                            | Do this                                                          |
| --------------------------------------- | ---------------------------------------------------------------- |
| Reject bad client input                 | `raise ValidationError(...)` from serializer or service          |
| Block an unauthorized action            | `raise PermissionDenied(...)`                                    |
| Return 404                              | `get_object_or_404(...)` or `raise Http404`                      |
| Return a domain-specific 4xx            | Define an `APIException` subclass with `status_code` + `default_code` |
| Handle a flaky external API             | `try/except` around the call site, retry or re-raise as domain exception |
| Process a batch where some rows fail   | Per-record `try/except`, log + count, continue                   |