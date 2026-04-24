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
│   └── urls.py
├── WIP/                 # parent app with sub-apps
│   ├── sheets/
│   │   ├── urls.py
│   │   ├── views.py
│   │   └── models.py
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

| I want to add…            | Where it goes                                      |
|---------------------------|----------------------------------------------------|
| A new feature area        | A new Django app at the root                       |
| A new sub-feature of WIP  | A new sub-app inside `WIP/`                        |
| Project-wide settings     | `CLRPLN/settings.py`                               |
| A reusable service        | `services.py` inside the relevant app              |
| A custom management cmd   | `<app>/management/commands/<name>.py`              |
