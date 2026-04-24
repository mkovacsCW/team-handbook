# Day One

A checklist to get you up and running. Work through it at your own pace — expect it to take most of your first day.

## Accounts & access

- [ ] GitHub — ask your manager to add you to the org
- [ ] Slack / team chat — join `#engineering` and your team's channel
- [ ] Ticketing system (Jira / Linear / etc.)
- [ ] VPN (if needed for internal services)
- [ ] Password manager

## Local environment setup

!!! info "TODO for the doc maintainer"
    Fill this section in with your actual setup steps. Example below.

### Backend

```bash
# Clone the repo
git clone git@github.com:your-org/your-repo.git
cd your-repo

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

### Frontend

```bash
cd Front-End
npm install
npm run dev
```

Open http://localhost:5173 — you should see the login page.

## Your first PR

To get familiar with the workflow without any pressure:

1. Pick a tiny issue tagged `good first issue` (or ask for one).
2. Read the [Pull Requests guide](../workflow/pull-requests.md).
3. Create a branch using GitHub's "Create a branch" button on the issue.
4. Make the change, push, and open a draft PR.
5. Ask your onboarding buddy to walk through it with you before you mark it ready for review.

!!! tip
    Your first PR is not about productivity — it's about learning our review process, CI, and deploy pipeline. Small and boring is perfect.

## Who to ask for what

| If you need…              | Ask…                        |
|---------------------------|-----------------------------|
| Dev environment help      | Your onboarding buddy       |
| Backend / Django question | `#backend` channel          |
| Frontend / React question | `#frontend` channel         |
| Access to a system        | Your manager                |
| HR / admin stuff          | (fill in)                   |
