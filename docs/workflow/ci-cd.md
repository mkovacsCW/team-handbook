# CI/CD

!!! info "Coming soon"
    This is planned but not yet implemented. Deploys are currently manual — see [Systems Development → Deploy](systems-development.md#7-deploy).

## The plan

We're planning to set up a CI/CD pipeline using GitHub Actions. When it lands, it will:

- **Run tests automatically** on every PR — no more relying on the author to remember
- **Block merges on failing tests** — if tests don't pass, the PR can't be merged
- **Auto-deploy to production** on merge to `main` — no manual deploy step

## What this means for you (today)

Not much changes until it's built. Keep merging and deploying the way you always have.

The one thing to keep in mind: **write your code as if tests will be run automatically in the future**. That means:

- Tests should pass locally without manual setup
- Tests shouldn't depend on your specific dev database state
- Tests shouldn't require secrets or external services that won't be available in CI

This way, when the pipeline does get set up, existing tests will "just work."

## When it lands

This page will be updated with the actual workflow files, how to interpret failures, how to trigger manual deploys, and what to do when CI goes red.
