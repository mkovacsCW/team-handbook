# Project Structure

We use an `apps/` pattern at the project root to group Django apps. Each app usually maps to a feature area (e.g., `UserAuth` for authentication). Some "parent" apps (like `WIP`) contain sub-apps for features within that module.

```
Back-End/
├── CLRPLN/              # project config
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── UserAuth/            # feature app
│   ├── management/commands/
│   ├── media/
│   ├── migrations/
│   ├── views.py
│   ├── models.py
│   ├── services.py      # or services/ folder if it grows
│   ├── utils.py         # or utils/ folder if it grows
│   └── urls.py
├── WIP/                 # parent app with sub-apps
│   ├── sheets/
│   │   ├── urls.py
│   │   ├── views.py
│   │   └── models.py
│   ├── services/        # services package (grew past one file)
│   │   ├── __init__.py
│   │   ├── answer_validation.py
│   │   └── sheet_submission.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── excel.py
│   │   └── dates.py
│   ├── migrations/
│   ├── misc_reports/
│   ├── system/
│   ├── reviewer_reports/
│   ├── urls.py
│   ├── views.py
│   └── apps.py
├── manage.py
├── .env
└── requirements.txt
```

## Naming

See the [quick reference table](index.md#quick-reference) on the Backend landing page.

## Where things go

| I want to add…              | Where it goes                                          |
|-----------------------------|--------------------------------------------------------|
| A new feature area          | A new Django app at the root                           |
| A new sub-feature of WIP    | A new sub-app inside `WIP/`                            |
| Project-wide settings       | `CLRPLN/settings.py`                                   |
| Business logic              | `services.py` (or `services/<concern>.py`) in the app  |
| A small reusable helper     | `utils.py` (or `utils/<topic>.py`) in the app          |
| A custom management command | `<app>/management/commands/<name>.py`                  |
| A scheduled cron job        | Management command + line in the `Dockerfile` crontab  |
| A custom exception          | `<app>/exceptions.py`                                  |
| A custom queryset/manager   | On the model in `models.py` (or a `managers.py`)       |

## When `services.py` / `utils.py` becomes a folder

Both follow the same rule. Start with a single file. Promote to a package once the file gets long or covers multiple concerns:

```
# Single file — fine when small
services.py

# Folder — once it grows
services/
├── __init__.py
├── answer_validation.py
└── sheet_submission.py
```

Name files after the **concern**, not the model. See [Services](services.md) and [Utils](utils.md) for details.