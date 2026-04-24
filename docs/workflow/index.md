# Workflow

How we ship code as a team, handle support, and (eventually) deploy.

## Pages in this section

- **[Systems Development](systems-development.md)** — how we build features start to finish, from design through deploy.
- **[Pull Requests](pull-requests.md)** — how we size, write, review, and merge PRs.
- **[Support Email Process](support-email.md)** — how we handle the shared inbox.
- **[CI/CD](ci-cd.md)** — planned automation (not yet built).

## Quick reference

| Situation                       | What to do                                                  |
|---------------------------------|-------------------------------------------------------------|
| Starting a new feature          | Read [Systems Development](systems-development.md). Sketch the data model before writing code. |
| Starting new work (general)     | Create a branch from the issue using GitHub's "Create a branch" button. |
| New UI component                | Prototype first with hardcoded data. Match the styling of the most recently deployed feature. |
| PR comment prefixes             | `nit:` `suggestion:` `question:` `blocking:`                |
| Request Changes vs Approve      | Request Changes only for genuine blockers. Approve with comments for everything else. |
| When to merge                   | Author merges after approvals + green CI. Squash merge. Delete branch. |
| Stale PR                        | If no activity after a few days, follow up or close.        |
| Support email you can't reply to | Leave it unread OR flag/star it. Never read-and-ignore.    |
