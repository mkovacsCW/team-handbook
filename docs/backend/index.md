# Backend

Django + DRF patterns and conventions.

!!! warning "Heads up: not every file follows these conventions"
    Parts of the backend predate this handbook. The conventions on the following pages describe **where we're going**, not always where every file currently is. Some patterns you'll run into:

    - **Views with business logic baked in** — instead of thin views calling services.
    - **Apps with no `services.py` at all** — logic lives wherever it landed.
    - **Massive `views.py` files** — hundreds (sometimes thousands) of lines of standalone function-based views, no `ViewSet`, no shared `get_queryset`, lots of repetition.
    - **Endpoints that skip serializers entirely** — manual `request.data` parsing, manual dict-building for responses, no validation layer.
    - **Email helpers still in `UserAuth/utils.py`** — instead of `CLRPLN/emails.py`.
    - **Inline HTML or raw SQL** in places where a serializer or ORM query would be cleaner.

    **The direction we're moving:**

    - **New code follows the handbook.** Thin views, business logic in services, helpers in utils, emails composed from `CLRPLN/emails.py` building blocks (see [Emails](emails.md)).
    - **When you touch an older file for unrelated work, you don't have to rewrite it in the same PR** (see [Clean as You Go](../workflow/general-principles.md#11-clean-as-you-go) — keep cleanup proportional). But prefer migrating it over patching it if the change is non-trivial — adding a third branch to a 200-line view is the moment to extract a service.
    - **Don't add new Postmark-template-based emails.** Existing ones can stay until they're touched. New emails use inline HTML composed from `CLRPLN/emails.py` (see [Emails](emails.md)).
    - **Over time, deprecate and delete legacy patterns** as they're replaced — old utility duplicates, dead management commands, single `services.py` files that should be folders.

    **Found a file, command, or model that looks unused or dead?** Don't just delete it — ping **Marton** or **Logan** first. The Dockerfile schedules ~20 cron jobs, some of them weekly or monthly, so a "clearly unused" management command might run at 3am on the first Tuesday of every month and cost real money if it stops.

## Pages in this section

- **[Project Structure](project-structure.md)** — how apps are organized.
- **[Models](models.md)** — fields, inheritance, mixins, Meta, methods.
- **[Querysets & Managers](querysets-managers.md)** — custom managers, chainable querysets.
- **[Serializers](serializers.md)** — purpose-specific serializers, optimization, validation.
- **[Views](views.md)** — FBVs, APIViews, ViewSets, when to use each.
- **[URLs](urls.md)** — structure, naming, REST conventions.
- **[Services](services.md)** — where business logic lives.
- **[Utils](utils.md)** — small reusable helpers.
- **[Exceptions & Error Handling](exceptions.md)** — what to raise, when to catch.
- **[Emails](emails.md)** — Postmark, building blocks, the DEBUG redirect, conventions.
- **[Management Commands](management-commands.md)** — custom commands and cron scheduling.

## Quick reference

| Type        | Convention                | Example                                    |
| ----------- | ------------------------- | ------------------------------------------ |
| Apps        | snake_case, plural        | `orders`, `user_profiles`                  |
| Models      | PascalCase, singular      | `Order`, `UserProfile`                     |
| Serializers | PascalCase + Serializer   | `OrderSerializer`, `OrderDetailSerializer` |
| Views       | PascalCase + ViewSet/View | `OrderViewSet`, `OrderListView`            |
| Services    | snake_case functions      | `create_order`, `close_eligible_jobs`      |
| Utils       | snake_case functions      | `previous_business_day`, `format_phone`    |
| Commands    | snake_case verb_noun      | `send_welder_reports`, `auto_clock_out`    |
| URLs        | kebab-case                | `/api/user-profiles/`                      |
| Constants   | SCREAMING_SNAKE_CASE      | `MAX_ORDER_ITEMS`                          |

!!! warning "Always commit migrations"
    Include any migrations you've generated when putting up pull requests. Do **not** leave migrations uncommitted.