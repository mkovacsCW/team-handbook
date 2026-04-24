# Week One

A checklist to get you up and running. Work through it at your own pace — expect it to take most of your first week.

## Accounts & access

- [ ] GitHub — ask Michelle to give you repository access if you haven't been given access already
- [ ] Ensure you have been added to the Microsoft Teams channels
- [ ] Ensure your outlook email is set up on your computer
- [ ] Ensure your internal VPN works (in case you need to access the company network when working out-of-office)
- [ ] Verify your computer has two-factor authentication

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
| Tasks/Issue Assignments        | Michelle                |
| Time-Off, WFH, Bereavemenet Requests   | Michelle                |
| HR Requests / Questions         | Michelle Haydary                   |
| IT Requests        | Andre Dacillo or Carl Vivo                 |
| General Opinion       | Any CLRPLN team member               |
