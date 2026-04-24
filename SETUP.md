# Setup Guide

This walks you from zero to a live docs site on GitHub Pages. Expect ~10 minutes.

**End result:** A public URL like `https://<your-org>.github.io/engineering-handbook/` that auto-rebuilds every time someone pushes to `main`.

---

## Before you start

You'll need:

- [x] A GitHub account (or org) where you can create a new repo
- [x] Git installed locally
- [x] Python 3.8+ installed locally (only for previewing changes; deploys happen on GitHub)

---

## Step 1: Create a new GitHub repo

1. Go to https://github.com/new
2. **Repository name:** `engineering-handbook` (or whatever feels right — `team-docs`, `dev-handbook`, etc.)
3. **Visibility:** Private is fine. The *repo* stays private; only the *built site* will be public.
4. **Do NOT** check "Add a README" — we're pushing one from this starter.
5. Click **Create repository**.

GitHub will show you a page with git commands. Ignore those, we'll use the ones below.

---

## Step 2: Push this starter to the repo

Unzip this starter somewhere on your machine, open a terminal in that folder, and run:

```bash
git init
git add .
git commit -m "Initial docs"
git branch -M main
git remote add origin git@github.com:YOUR-ORG/engineering-handbook.git
git push -u origin main
```

Replace `YOUR-ORG/engineering-handbook` with your actual org and repo name.

!!! tip "HTTPS vs SSH"
    If you don't have SSH keys set up with GitHub, use the HTTPS URL instead:
    `git remote add origin https://github.com/YOUR-ORG/engineering-handbook.git`

---

## Step 3: Enable GitHub Pages

1. In the repo on GitHub, go to **Settings → Pages**.
2. Under **Build and deployment → Source**, select **GitHub Actions**.

   (Not "Deploy from a branch" — we're using the workflow file included in this starter.)

That's it for Pages config. The GitHub Action in `.github/workflows/deploy.yml` will handle the rest.

---

## Step 4: Trigger the first deploy

The workflow runs automatically on every push to `main`. You already pushed in Step 2, so it's probably already running. Check:

1. Go to the **Actions** tab in your repo.
2. You should see a "Deploy docs" workflow running (or completed).
3. Once it's green, your site is live at `https://YOUR-ORG.github.io/engineering-handbook/`.

First build takes ~1 minute. Subsequent builds are faster (dependencies are cached).

!!! info "First deploy failed?"
    Most common cause: you forgot to set **Source: GitHub Actions** in Step 3. Do that, then re-run the workflow from the Actions tab.

---

## Step 5: Verify it works

Open the URL. You should see:

- A landing page with card grid navigation
- Tabs at the top: Onboarding, Workflow, Backend, Frontend
- Working search (top right)
- A dark mode toggle

Click around. Try the search. Everything should feel snappy.

---

## You're live. Now what?

### Edit docs locally (optional but recommended)

To preview changes before pushing:

```bash
pip install -r requirements.txt
mkdocs serve
```

Opens at `http://127.0.0.1:8000` with hot reload.

### Fill in the placeholders

Three pages have `TODO` callouts I left for you:

- `docs/onboarding/day-one.md` — your actual env setup and access checklist
- `docs/onboarding/glossary.md` — internal terms (CLRPLN, WIP, etc.)
- `docs/onboarding/codebase-tour.md` — adjust the folder structure to match reality

Edit them, commit, push. Auto-deploys in ~1 min.

### Share with the team

Your URL: `https://YOUR-ORG.github.io/engineering-handbook/`

Drop it in teams, pin it in your onboarding channel, put it in your welcome email to new hires.

---

## Common tasks

### Adding a new page

1. Create a `.md` file in the appropriate folder under `docs/`.
2. Add an entry to the `nav:` section of `mkdocs.yml`.
3. Commit and push. Auto-deploys.

### Reorganizing the sidebar

Edit the `nav:` section of `mkdocs.yml`. The order in the file = the order in the sidebar.

### Using a custom domain (e.g. `docs.yourcompany.com`)

1. In repo **Settings → Pages**, add your custom domain.
2. GitHub tells you to add a CNAME DNS record. Do that with your DNS provider.
3. Wait for DNS to propagate (usually < 10 min, sometimes an hour).
4. Enable "Enforce HTTPS" once the cert is ready.

### Adding formatting features

This starter already has callouts, code tabs, mermaid diagrams, task lists, and more. See the **Writing tips** section of `README.md` for syntax.

---

## Troubleshooting

**Workflow fails with "pages not enabled"** → You missed Step 3. Go to Settings → Pages and set source to GitHub Actions.

**Workflow fails with Python errors** → Open the failing Actions run, expand the "Build site" step, read the actual error. Usually a typo in `mkdocs.yml` or a markdown file.

**Site loads but styles are broken** → Hard refresh with Ctrl+F5 / Cmd+Shift+R. If still broken, check that `site_url` in `mkdocs.yml` matches your actual URL.

**404 on a page that exists** → MkDocs URLs don't include `.md`. `docs/backend/models.md` → `/backend/models/`. Check the built URL, not the filename.

**"strict" mode failing on warnings** → The workflow uses `mkdocs build --strict` which fails on any warning (broken links, missing files in nav, etc.). This is on purpose — catches issues before they hit production. Fix the warning, or remove `--strict` from `.github/workflows/deploy.yml` if you want to be more permissive.

---

## One thing about privacy

Your **repo** is private. The **built site** is public (this is just how GitHub Pages works on Free/Pro/Team plans).

This starter is configured so the public site has **zero links back to the repo**:

- `repo_url` and `edit_uri` in `mkdocs.yml` are commented out (don't uncomment them)
- No "Edit this page" buttons
- No GitHub icon in the top right
- No repo name visible in the footer

So visitors see a clean docs site with no trail back to your source code. If you ever want to change this and add a "Edit this page" link, uncomment those two lines in `mkdocs.yml`.

---

## Next steps

Once you've got this running, the [main README](./README.md) covers:

- Writing tips (callouts, tabs, mermaid diagrams)
- Syntax for formatted code blocks
- Plugins you might want to add later (like "last updated" dates)

Questions? The [MkDocs Material docs](https://squidfunk.github.io/mkdocs-material/) are excellent and searchable.
