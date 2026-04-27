# Backend

Django + DRF patterns and conventions.

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