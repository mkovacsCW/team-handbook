# URLs

## Structure & Organization

Group related endpoints with comments and keep a logical order:

```python
urlpatterns = [
    # --- Workers ---
    path('workers/', views.get_workers, name='get_workers'),
    path('workers/<int:pk>/', views.get_worker, name='get_worker'),
    path('workers/<int:pk>/time-entries/', views.get_worker_time_entries, name='worker_time_entries'),

    # --- Crews ---
    path('crews/', views.get_crews, name='get_crews'),
]
```

## Naming Conventions

- Use **kebab-case** for URLs: `work-orders/`, not `work_orders/`
- Use **snake_case** for `name=`: `name='get_work_orders'`
- Be consistent with param names: pick `pk` or `<model>_id` and stick with it
- Avoid verbs in URLs when possible — let HTTP methods do the work

## REST-Style URLs

Prefer resource-based paths over action-based:

```python
# Good (RESTful)
path('articles/', views.list_articles),                   # GET
path('articles/', views.create_article),                  # POST
path('articles/<int:pk>/', views.get_article),            # GET

# Avoid
path('articles/create/', views.create_article)
path('articles/get-all/', views.list_articles)
```

## Using DRF Routers (ViewSets)

For CRUD-heavy resources, routers reduce boilerplate:

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'articles', ArticleViewSet, basename='article')

urlpatterns = [
    path('', include(router.urls)),
    # Custom non-CRUD endpoints
    path('articles/export/', views.export_articles, name='export_articles'),
]
```

## Nested Resources

Keep nesting shallow (max 2 levels):

```python
# Good
path('crews/<int:crew_id>/workers/', views.get_crew_workers),

# Avoid deep nesting
path('companies/<int:company_id>/crews/<int:crew_id>/workers/<int:worker_id>/entries/')
```

## Quick Tips

- Always set `name=` for reverse URL lookups.
- Use `include()` to split large apps into sub-files.
- Put specific routes before generic ones (`workers/unassigned/` before `workers/<int:pk>/`).
- Use `path()` over `re_path()` unless you need regex.
- Keep imports clean — import views module, not individual functions.
