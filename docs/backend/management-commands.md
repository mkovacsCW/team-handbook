# Management Commands

Management commands are scripts you run with `python manage.py <command>`. We use them for:

- **Scheduled jobs** run by cron (weather syncs, KPI checks, allocation reminders).
- **Data imports / one-off backfills** (`import_data`, populating a new field).
- **Operational tools** (`auto_clock_out`, `check_job_closing`).

If a piece of logic needs to run on a schedule **or** be triggered manually by a developer in production, it belongs in a management command.

## File layout

```
myapp/
└── management/
    ├── __init__.py
    └── commands/
        ├── __init__.py
        └── send_welder_reports.py
```

The filename **is** the command name. `commands/send_welder_reports.py` → `python manage.py send_welder_reports`.

## Writing a command

```python
# wip/management/commands/check_job_closing.py
from django.core.management.base import BaseCommand
from django.utils import timezone

from wip.services.job_closing import close_eligible_jobs


class Command(BaseCommand):
    help = "Close jobs that have met all closing criteria."

    def add_arguments(self, parser):
        parser.add_argument(
            '--dry-run',
            action='store_true',
            help="Show what would be closed without making changes.",
        )
        parser.add_argument(
            '--days',
            type=int,
            default=7,
            help="Look back this many days for eligible jobs.",
        )

    def handle(self, *args, **options):
        dry_run = options['dry_run']
        days = options['days']

        self.stdout.write(f"Checking jobs from the last {days} days...")
        if dry_run:
            self.stdout.write(self.style.WARNING("DRY RUN — no changes will be made."))

        result = close_eligible_jobs(days=days, dry_run=dry_run)

        # End-of-run report — what happened.
        self.stdout.write(self.style.SUCCESS(
            f"Done. closed={result['closed']} "
            f"skipped={result['skipped']} "
            f"errors={result['errors']}"
        ))
```

The actual logic lives in a service (`close_eligible_jobs`). The command is a thin wrapper that handles arguments, prints output, and reports results — same principle as keeping views thin.

## End-of-run reports

Every command that does work should print a **summary line** at the end: how many records were processed, updated, skipped, and errored. Cron output goes to `/var/log/cron.log` and that summary is what we look at first when something seems off.

```
Done. closed=5 skipped=2 errors=1
```

If there are errors worth seeing, print details before the summary using `self.style.ERROR(...)`.

## Scheduling a command (cron)

Cron jobs live in the `Dockerfile`. After adding a new command, append a line to the crontab section:

```dockerfile
RUN echo "0 8 * * * /usr/local/bin/python /app/manage.py check_job_closing >> /var/log/cron.log 2>&1" \
    | tr -d '\r' >> /etc/cron.d/mycron
```

A few notes on the existing crontab worth knowing:

- Times are **America/New_York** (set via `ENV TZ` in the Dockerfile).
- The `tr -d '\r'` strips Windows line endings — leave it in even if you're on a Mac.
- Output is appended to `/var/log/cron.log`. `tail -f` that file when debugging.
- For "first Tuesday of the month" or other complex schedules, gate inside the cron line (see `send_allocation_reminder_on`) **or** let the command self-gate by checking the date and exiting early.

After editing the Dockerfile, the container needs to be rebuilt for the new schedule to take effect.

## General best practices

- **Put the real work in a service.** The command should mostly parse arguments, call the service, and print results.
- **Always support `--dry-run`** for anything that writes data or sends external messages (emails, API calls). It saves you from a bad backfill.
- **Idempotent by default.** Running a command twice should not double-send emails, double-create rows, or double-charge anything. Check state before acting.
- **Print a summary at the end.** `created=X updated=Y skipped=Z errors=N`. This is what we look at in `cron.log`.
- **Use `self.stdout.write` and `self.style.*`**, not `print()`. They respect Django's verbosity settings and the colored output helps in terminals.
- **Catch and log exceptions per record**, not per run, when processing batches. One bad row shouldn't kill 999 good ones.
- **Be explicit about timezones.** `timezone.now()` not `datetime.now()`.

## Common patterns

| Use case                                | Pattern                                                          |
| --------------------------------------- | ---------------------------------------------------------------- |
| Scheduled email/report                  | Command → service → email util. Cron line in Dockerfile.         |
| Data backfill (one-off)                 | Command with `--dry-run` and `--batch-size`. Run, then delete.   |
| Sync from external API                  | Command → service. Idempotent. Log per-record errors, summarise. |
| "First Tuesday of month" type schedules | Self-gate inside the command via `date.today()` checks.          |