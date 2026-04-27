# Week One

A checklist to get you up and running. Work through it at your own pace — expect it to take most of your first week.

## Accounts & access

- [ ] GitHub — ask Michelle to give you repository access if you haven't been given access already
- [ ] Ensure you have been added to the Microsoft Teams channels
- [ ] Ensure your outlook email is set up on your computer
- [ ] Ensure your internal VPN works (in case you need to access the company network when working out-of-office)
- [ ] Verify your computer has two-factor authentication

## Required software

Install the following on your machine before moving on to local environment setup:

- [ ] **Node.js** — for the frontend
- [ ] **Python 3.11** — for the backend
- [ ] **Git** — for version control
- [ ] **An IDE** — VS Code is recommended, but Neovim and other editors are also fine
- [ ] **PostgreSQL 17.5** — this is the version we standardize on, so please match it
- [ ] **pgAdmin 4** — for managing your local database

You'll also need the **ODBC driver** installed. IT has to do this for you — reach out to Marton or Logan and they'll help you put in the request.

## Local environment setup

### Clone the repository
```bash
# Clone the repo
git clone [REPO LINK]
cd CLRPLN
```

### Backend
```bash
# Set up Python environment
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Copy env template and fill in values (ask a teammate for dev creds)
cp .env.example .env

# Run migrations
python manage.py migrate

# Start the dev server
python manage.py runserver
```

### Import a database sample

Ask Marton or Logan for a recent database sample. Once you have it, import it through pgAdmin into your local database.

After the import finishes, disable email notifications and 2FA for every user on your local copy — this keeps your local environment from sending real emails or locking you out behind a 2FA prompt you can't satisfy. Run this in pgAdmin's query tool:

```sql
update "UserAuth_customuser"
set email_notifications = False,
    two_factor_enabled = False,
    enforce_2fa = False;
```

Then import the permission set so users have the right access in your local environment:

```bash
python manage.py import_permissions
```

Finally, reset every user's password to a known value so you can log in as anyone for debugging:

```bash
python manage.py reset_all_passwords --dev-default
```

This sets every account's password to `12345678`. Only ever run this against your local database — never against staging or production.

### Frontend

```bash
cd Front-End
npm install
npm run dev
```

Open http://localhost:5173 — you should see the login page.

## Your first PR

To get familiar with the workflow without any pressure:

1. Michelle will assign a tiny issue tagged `good first issue` (or ask for one if you haven't been assigned).
2. Read the [Pull Requests guide](../workflow/pull-requests.md).
3. Create a branch using GitHub's "Create a branch" button on the issue.
4. Make the change, push, and open a draft PR.
5. Ask your onboarding buddy to walk through it with you before you mark it ready for review.

!!! tip
    Your first PR is not about productivity — it's about learning our review process, CI, and deploy pipeline. Small and boring is perfect. Please ask as many questions as possible. Ask any question, even if you think it might be a stupid one. Every question is welcome.

## Who to ask for what

| If you need…              | Ask…                        |
|---------------------------|-----------------------------|
| Dev environment help      | Your onboarding helper       |
| Backend / Django question | Logan          |
| Frontend / React question | Marton        |
| Access to a system        | Michelle                |
| High Level Project Questions (e.g requirement clarification)        | Michelle                |
| Tasks/Issue Assignments        | Michelle                |
| Time-Off, WFH, Bereavement Requests   | Michelle                |
| HR Requests / Questions         | Michelle Haydary                   |
| IT Requests        | Andre Dacillo or Carl Vivo                 |
| General Coding Help or Opinion       | Any CLRPLN team member               |