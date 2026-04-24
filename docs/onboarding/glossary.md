# Glossary

Internal terms, project codenames, and acronyms you'll see in the codebase.

!!! info "TODO for the doc maintainer"
    Fill in real definitions. Examples below based on what showed up in your existing docs.

## Project / module names

| Term      | Meaning                                                                 |
|-----------|-------------------------------------------------------------------------|
| **CLRPLN**   | The Django project package — contains settings, root URLs, WSGI/ASGI. |
| **WIP**      | Work-In-Progress reporting module. Parent app with several sub-apps.  |
| **UserAuth** | Authentication & user management app.                                 |

## Domain terms

| Term       | Meaning                                                                  |
|------------|--------------------------------------------------------------------------|
| **Locate** | A request to mark underground utilities at a job site. Has an expiration date and lifecycle (Requested → Active → Expiring Soon → Expired / Terminated). |
| **Job**    | A project/contract. Locates, sheets, and reports belong to a Job.        |
| **Sheet**  | (fill in)                                                                |
| **Crew**   | A group of workers assigned to a Job.                                    |

## Tech stack shorthand

| Term     | Meaning                                           |
|----------|---------------------------------------------------|
| **DRF**  | Django REST Framework — the API layer.            |
| **FBV**  | Function-Based View.                              |
| **CBV**  | Class-Based View.                                 |
| **N+1**  | A query pattern where you make 1 query plus N more (one per row). Usually a performance bug. Fix with `select_related` / `prefetch_related`. |
| **SPA**  | Single-Page Application (our React frontend).     |
