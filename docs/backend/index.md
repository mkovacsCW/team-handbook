# Backend

Django + DRF patterns and conventions.

## Pages in this section

- **[Project Structure](project-structure.md)** — how apps are organized.
- **[Models](models.md)** — fields, inheritance, mixins, Meta, methods.
- **[Serializers](serializers.md)** — purpose-specific serializers, optimization, validation.
- **[Views](views.md)** — FBVs, APIViews, ViewSets, when to use each.
- **[URLs](urls.md)** — structure, naming, REST conventions.

## Quick reference

| Type        | Convention                | Example                                    |
| ----------- | ------------------------- | ------------------------------------------ |
| Apps        | snake_case, plural        | `orders`, `user_profiles`                  |
| Models      | PascalCase, singular      | `Order`, `UserProfile`                     |
| Serializers | PascalCase + Serializer   | `OrderSerializer`, `OrderDetailSerializer` |
| Views       | PascalCase + ViewSet/View | `OrderViewSet`, `OrderListView`            |
| Services    | snake_case + `_service`   | `order_service.py`                         |
| URLs        | kebab-case                | `/api/user-profiles/`                      |
| Constants   | SCREAMING_SNAKE_CASE      | `MAX_ORDER_ITEMS`                          |

!!! warning "Always commit migrations"
    Include any migrations you've generated when putting up pull requests. Do **not** leave migrations uncommitted.
