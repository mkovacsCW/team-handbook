# Pull Requests

A guide for how our team writes, reviews, and merges pull requests.

## PR Size & Scope

- **Keep PRs small and focused.** Each PR should represent one logical change — a single feature, bug fix, or refactor. If you find yourself writing a novel in the description, the PR is probably too big.
- **Break up large changes.** If a change touches many files or systems, consider splitting it into stacked or sequential PRs. This makes review faster and reduces the risk of sneaking in bugs.

## Titles & Descriptions

- **Write clear, descriptive titles.** The title should tell a reviewer what the PR does at a glance. Prefer `Fix race condition in session cleanup job` over `fix bug`.
- **Use the description to give context.** Every PR description should cover:
    - **What** changed
    - **Why** it changed (link to the issue/ticket)
    - **How to test** or verify the change
    - **Risks or trade-offs**, if any
- **Link the relevant issue or ticket.** Don't make reviewers guess what this is for.

## Commit Hygiene

- **Write meaningful commit messages.** Each commit message should describe what it does and why. Avoid `wip`, `asdf`, `fix`, etc.
- **Squash before merging** (or use squash-merge). The main branch history should read as a clean series of logical changes, not a stream of consciousness.

## Branch Strategy

- **Use GitHub's "Create a branch" feature** from the issue. This automatically names the branch using the issue number and title (e.g., `42-fix-login-timeout`), keeping things consistent and traceable without extra effort.
- **Branch off of `main`** and target your PR back to it unless you're working on a long-lived feature branch.
- **Keep your branch up to date.** Rebase or merge `main` into your branch before requesting review to avoid painful merge conflicts later.

## Draft PRs

- Use draft PRs when you want early feedback on an approach, or to give the team visibility into work in progress.
- Make it clear in the description what kind of feedback you're looking for (architecture, naming, general direction, etc.)

## Screenshots & Visuals

!!! tip "UI changes need visuals"
    For any UI change, include before/after screenshots or a short screen recording. This makes review dramatically faster and catches visual regressions early.

## Requesting Review

- **Tag the right people.** Use code owners where possible, and tag domain experts for areas you're less familiar with.
- **Don't tag the entire team.** One or two reviewers is usually enough.
- **Respect people's time.** If your PR is large or complex, give reviewers a heads up or walk them through it.

## Reviewing PRs

**Review within one business day.** Timely reviews keep the team moving. If you can't review in time, let the author know.

**Be clear about severity.** Prefix comments to signal intent:

| Prefix        | Meaning                                         |
|---------------|-------------------------------------------------|
| `nit:`        | Style or preference, non-blocking               |
| `suggestion:` | Take it or leave it                             |
| `question:`   | I want to understand this better                |
| `blocking:`   | This needs to change before merge               |

**Be kind and constructive.** Ask questions rather than make demands. Assume good intent. Critique the code, not the person.

**Approve with comments vs. Request Changes** — use "Request Changes" only when something genuinely needs to change before merge. Use "Approve" with comments for nits and suggestions.

## Don't Let PRs Go Stale

- If a PR has been open for more than a few days without activity, the author should follow up or close it.
- Reviewers should flag if they're blocked or need more context rather than silently ignoring a review request.

## Merging

- **The author merges** after receiving the required approvals and CI is green.
- **Squash merge** into `main` to keep history clean (unless the team has agreed on a different strategy).
- **Delete the branch after merging.** Keep the repo tidy.

---

!!! note "Living document"
    If something here isn't working for the team, speak up and we'll adjust.
