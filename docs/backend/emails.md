# Emails

We send a lot of emails - confirmations, reminders, scheduled reports, allocation prompts, welder reports, and more. This page covers how we send them today, where we're heading, and the conventions to follow either way.

We use [Postmark](https://postmarkapp.com/) as our email provider, via the `postmarker` Python client.

## Where email code lives

### Today

Email helpers currently live in `UserAuth/utils.py` — a `send_email()` and `send_template_email()` pair that wraps Postmark. Specific emails (confirmations, reminders) live in `<app>/emails.py` files inside the apps that send them, and import the helpers from there.

### Where we're heading (don't refactor everything yet)

- `CLRPLN/emails.py` — shared building blocks: HTML wrapper, info table, data table, paragraph helper, the `send_postmark_email()` wrapper, `COLORS` palette.
- `<app>/emails.py` — sub-app-specific emails. Imports the shared helpers from `CLRPLN.emails` and composes them into specific emails (`send_allocation_confirmation_email`, `send_welder_report_email`, etc.).

```
CLRPLN/
└── emails.py            # shared building blocks (wrapper, tables, send helper)

monthly_allocation/
└── emails.py            # imports from CLRPLN.emails, defines specific emails

locates/
└── emails.py            # same — its own emails, shared building blocks
```

The goal is **one place to change the visual style, the safety nets, and the "how we talk to Postmark" code**. Sub-apps should rarely reach into Postmark directly.

!!! note "Don't bulk-refactor"
    Move email code into this structure as you touch it, not all at once. The current `UserAuth/utils.py` setup will continue to work — new emails should follow the new pattern, and old ones can be migrated when there's a reason to be in that file anyway.

## Two ways to send

### Inline HTML (preferred going forward)

We build the HTML in Python and pass it straight to Postmark. The styling, layout, and logic all live in our codebase, in version control, reviewable in PRs.

```python
# CLRPLN/emails.py (sketch)
from django.conf import settings
from postmarker.core import PostmarkClient
import logging

logger = logging.getLogger(__name__)

COLORS = {
    'primary': '#1e3a5f',
    'success': '#28a745',
    'danger': '#dc3545',
    'light_bg': '#f8f9fa',
    'border': '#e0e0e0',
    'text': '#333333',
    'text_light': '#666666',
}


def send_postmark_email(to_email, subject, html_body, cc_emails=None, bcc_emails=None, attachments=None):
    """Send an email via Postmark.

    In DEBUG mode, redirects to settings.DEBUG_EMAIL_REDIRECT to prevent
    accidentally emailing real users from local/staging environments.
    Returns (success: bool, message: str).
    """
    try:
        if settings.DEBUG:
            original_to = to_email
            to_email = settings.DEBUG_EMAIL_REDIRECT
            cc_emails = None
            bcc_emails = None
            subject = f"[DEBUG] {subject} (Original: {original_to})"
            logger.info("DEBUG: redirected email from %s to %s", original_to, to_email)

        postmark = PostmarkClient(server_token=settings.POSTMARK['SERVER_TOKEN'])
        email_data = {
            'From': 'systems@clrpln.ca',
            'To': to_email,
            'Subject': subject,
            'HtmlBody': html_body,
        }
        if cc_emails:
            email_data['Cc'] = ','.join(cc_emails) if isinstance(cc_emails, list) else cc_emails
        if bcc_emails:
            email_data['Bcc'] = ','.join(bcc_emails) if isinstance(bcc_emails, list) else bcc_emails
        if attachments:
            email_data['Attachments'] = attachments

        response = postmark.emails.send(**email_data)
        if response.get('ErrorCode') == 0 and response.get('Message') == 'OK':
            return True, "Email sent successfully"
        return False, f"Postmark error: {response.get('Message')}"

    except Exception as e:
        logger.error("Exception sending email: %s", e)
        return False, f"Exception sending email: {e}"


def email_wrapper(header_title, header_subtitle, content):
    """Outer HTML scaffold — header, body, footer. Pass content (HTML string) for the body."""
    # ... returns the wrapper HTML around `content`


def info_table(rows):
    """Build a label/value info table. ``rows`` is a list of (label, value) or (label, value, value_style) tuples."""
    # ...


def paragraph(text, margin="0 0 15px 0"):
    """Standard paragraph with house typography."""
    # ...
```

A specific email composes those helpers:

```python
# monthly_allocation/emails.py
from datetime import datetime
from CLRPLN.emails import (
    COLORS, send_postmark_email, email_wrapper, info_table, paragraph,
)


def send_allocation_confirmation_email(user, employee_no, allocation_month, allocations):
    """Confirmation email after a successful allocation submission."""
    if not user or not user.email:
        return False, "No user email found"

    month_name = allocation_month.strftime("%B %Y")
    subject = f"Monthly Allocation Submitted - {month_name}"

    table = info_table([
        ("Employee", f"{user.name} ({employee_no})"),
        ("Month", month_name),
        ("Jobs Allocated", str(len(allocations))),
        ("Submitted At", datetime.now().strftime('%Y-%m-%d %H:%M')),
    ])

    content = f"""
        {paragraph(f"Hello {user.name},")}
        {paragraph(f'Your monthly allocation has been <span style="color: {COLORS["success"]}; font-weight: bold;">&#10003; SUBMITTED</span>.')}
        {table}
        {paragraph("Thank you!", "0")}
    """

    html_body = email_wrapper("Monthly Allocation Confirmation", month_name, content)
    return send_postmark_email(user.email, subject, html_body)
```

### Postmark templates (legacy)

Older emails use Postmark's hosted templates — the HTML lives in the Postmark dashboard, we send a `template_alias` and a `TemplateModel` dict.

```python
from UserAuth.utils import send_template_email

send_template_email(
    to_email=user.email,
    template_alias='order-confirmation',
    template_data={'name': user.name, 'order_id': order.id},
)
```

Don't use this for new emails. Existing template-based emails can stay until there's a reason to touch them. The reason we're moving away: template HTML isn't in version control, isn't reviewable in PRs, and changes go live the moment someone clicks Save in the Postmark dashboard.

## The DEBUG redirect — always use it

`send_postmark_email` checks `settings.DEBUG` and reroutes the recipient to `settings.DEBUG_EMAIL_REDIRECT` (usually whichever developer is currently testing). This is the single biggest safety net we have against "I just emailed 400 real users from my laptop".

```python
# CLRPLN/settings.py
DEBUG_EMAIL_REDIRECT = os.environ.get('DEBUG_EMAIL_REDIRECT', 'developer@clearwaygroup.com')
```

**Every** new send function must go through `send_postmark_email` (or another helper that goes through it). Don't call `PostmarkClient.emails.send(...)` directly from a sub-app — that bypasses the safety net.

If you find yourself wanting to bypass it for a "real" test, change `DEBUG` in your environment, don't remove the check.

## Notifications vs. transactional emails

Two flavors of email, with different rules:

| Type            | Examples                                          | Respect `email_notifications`? |
| --------------- | ------------------------------------------------- | ------------------------------ |
| **Notification** | Allocation reminders, KPI nudges, status updates | **Yes** — check the flag       |
| **Transactional** | Password reset, 2FA code, allocation receipts after the user just acted | **No** — always send           |

The rule of thumb: if the user **just took an action** and this email confirms or completes it, send it regardless of preferences. If we're prompting them or telling them about something they didn't directly trigger, check the flag first.

```python
# Notification — check the flag
def send_allocation_reminder(user):
    if not user.email_notifications:
        return False, "User has notifications disabled"
    # ...

# Transactional — always send
def send_allocation_confirmation_email(user, ...):
    # User just submitted; always confirm. No flag check.
    # ...
```

## Recipients & attachments

**Recipient lists** generally come from a queryset filter — by province, role, or which workers haven't submitted something yet. Build that list in the calling service or management command, not inside the email function. The email function takes a user (or list) and sends; it shouldn't be querying who to send to.

**Attachments** for Postmark are dicts with `Name`, `Content` (base64), and `ContentType`:

```python
import base64

with open(report_path, 'rb') as f:
    encoded = base64.b64encode(f.read()).decode()

attachments = [{
    'Name': 'allocation_report.xlsx',
    'Content': encoded,
    'ContentType': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
}]

send_postmark_email(user.email, subject, html_body, attachments=attachments)
```

For larger reports (Excel allocation exports, welder reports), the file is built first by a service or util, then handed to the email function as an attachment. Building reports inside the email function couples two concerns that should stay separate.

## Outlook compatibility

A lot of our recipients read mail in Outlook, which is unforgiving:

- **Inline styles only.** No `<style>` blocks, no external CSS. Every `<td>` gets its own `style="..."`.
- **Tables for layout.** Flexbox and grid don't render. Nest tables.
- **No web fonts.** Stick to Arial / sans-serif.
- **MSO conditional comments** for Outlook-specific tweaks: `<!--[if mso]>...<![endif]-->`.
- **Test in Outlook** before shipping anything visually new — Gmail/Apple Mail will flatter you and Outlook will not.

The shared `email_wrapper` already handles most of this. New components should follow the same pattern — inline styles, tables, no fancy CSS.

## General best practices

- **All sends go through `send_postmark_email`.** Don't talk to `PostmarkClient` directly from sub-apps.
- **Check `email_notifications` for notification-style emails.** Skip the check for transactional ones.
- **Bail early on missing emails.** `if not user.email: return False, "..."` — log it, don't crash.
- **Log failures, don't crash callers.** Email failures shouldn't break the management command or API request that triggered them. Return `(success, message)` and let the caller decide what to do.
- **One email function per use case.** Named after what it does: `send_allocation_confirmation_email`, not `send_email_2`.
- **Build attachments outside the email function.** Email functions take ready-made attachments; they don't generate Excel files.
- **Don't put HTML strings in services or views.** They live in `emails.py`. A service calls `send_allocation_confirmation_email(user, ...)`, not a 200-line HTML template.
- **Always test with `DEBUG=True` first.** Watch the redirected email arrive at `DEBUG_EMAIL_REDIRECT` before flipping anything to prod.

## Quick reference

| You want to…                              | Use                                                            |
| ----------------------------------------- | -------------------------------------------------------------- |
| Send a new email type                     | New function in `<app>/emails.py`, compose helpers from `CLRPLN/emails.py` |
| Build the HTML body                       | `email_wrapper(...)` + `info_table(...)` + `paragraph(...)`    |
| Actually send                             | `send_postmark_email(...)` — never `PostmarkClient` directly   |
| Add a developer to receive DEBUG mail     | Set `DEBUG_EMAIL_REDIRECT` in your local env                   |
| Send a scheduled email                    | Management command → `<app>/emails.py` send function → cron line in Dockerfile |
| Attach a generated file                   | Build the file in a service/util, pass bytes as base64 attachment |
| Use an existing Postmark template         | `send_template_email` from `UserAuth/utils.py` (legacy — avoid for new code) |