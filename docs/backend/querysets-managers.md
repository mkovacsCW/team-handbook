# Querysets & Managers

Custom managers and querysets let you put **reusable query logic** on the model itself, instead of repeating the same filters in every view.

We mostly use the default manager. Reach for a custom one when:

- You're filtering the same way in lots of places (`is_deleted=False`, `company=request.user.company`).
- You want a clean API for special states (`User.objects.deleted_only()`).
- You need to override default behaviour globally (e.g. soft-delete).

## A real example: soft-deleted users

This is the pattern we use on `CustomUser` — the default queryset hides soft-deleted rows, with explicit methods to reach them when needed:

```python
from django.contrib.auth.models import UserManager


class CustomUserManager(UserManager):
    def get_queryset(self):
        # By default, exclude deleted users
        return super().get_queryset().filter(is_deleted=False)

    def all_with_deleted(self):
        # Method to get all users including deleted ones
        return super().get_queryset()

    def deleted_only(self):
        # Method to get only deleted users
        return super().get_queryset().filter(is_deleted=True)

    def get_by_natural_key(self, email):
        # Include deleted users when resolving natural keys.
        # This is crucial for data migrations.
        return self.all_with_deleted().get(email=email)


class CustomUser(AbstractUser):
    # ... fields ...
    is_deleted = models.BooleanField(default=False)
    deleted_at = models.DateTimeField(null=True, blank=True)

    objects = CustomUserManager()
```

Now `CustomUser.objects.all()` automatically excludes deleted rows everywhere, and admin/migration code can opt in via `all_with_deleted()`.

!!! warning "Overriding `get_queryset()` is a big hammer"
    Anything that hits `Model.objects` — admin lists, migrations, `select_related` from other models — will get the filtered queryset. If something genuinely needs the full set, it must use `all_with_deleted()` (or whatever escape hatch you provide). When in doubt, prefer a chainable queryset method (below) over overriding the default.

## Custom QuerySets (chainable)

If you want filters that **chain** (`.active().for_company(c).recent()`), use a custom `QuerySet` and expose it `as_manager()`:

```python
from django.db import models
from django.utils import timezone
from datetime import timedelta


class OrderQuerySet(models.QuerySet):
    def active(self):
        return self.filter(status='active')

    def for_company(self, company):
        return self.filter(company=company)

    def recent(self, days=30):
        cutoff = timezone.now() - timedelta(days=days)
        return self.filter(created_at__gte=cutoff)


class Order(models.Model):
    # ... fields ...
    objects = OrderQuerySet.as_manager()
```

Usage chains like normal querysets:

```python
Order.objects.for_company(request.user.company).active().recent()
```

This is usually what you want. Custom *managers* can't be chained off each other; custom *querysets* can.

## When to add a manager method

Ask: "Am I writing this same `.filter(...)` in three or more places?" If yes, lift it onto the queryset. If no, just write the filter inline — premature managers add indirection without payoff.

## General best practices

- **Prefer custom QuerySets over custom Managers** unless you need to override `get_queryset()` itself.
- **Don't shadow the default manager silently.** If overriding `get_queryset()`, always provide an escape hatch (`all_with_deleted()`).
- **Keep manager methods query-only.** No writes, no side effects — those are services.
- **Use `as_manager()`** so you don't have to write the manager class by hand.
- **Name methods like English.** `.active()`, `.for_company(c)`, `.due_today()` — not `.get_active_records()`.

## Quick reference

| You want…                                       | Use…                                  |
| ----------------------------------------------- | ------------------------------------- |
| Globally hide some rows (soft delete)           | Custom Manager overriding `get_queryset` |
| Reusable, chainable filters                     | Custom QuerySet + `as_manager()`      |
| One-off filter used in one view                 | Just write the filter inline          |
| A method that creates/updates rows              | Service function, not manager         |