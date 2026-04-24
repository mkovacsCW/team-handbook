# Systems Development

A walkthrough of how we build features from idea to production. This is the "big picture" glue that ties together our backend and frontend patterns — read this once when you join, and refer back when starting a large feature.

Most features don't need all of these steps. A small bug fix or a one-line tweak skips most of this. But a substantial feature — new database model, new endpoints, new UI — benefits from thinking through each stage.

## The phases

1. [Design and planning](#1-design-and-planning)
2. [UI prototyping](#2-ui-prototyping)
3. [Backend planning](#3-backend-planning)
4. [Frontend planning](#4-frontend-planning)
5. [Building incrementally](#5-building-incrementally)
6. [PR and review](#6-pr-and-review)
7. [Deploy](#7-deploy)

---

## 1. Design and planning

Before any code is written, get clarity on what you're building and why.

**Start from the problem, not the solution.** Write down what the user is trying to do, what's currently missing, and what "done" looks like. A paragraph is fine. If you can't summarize the problem in a paragraph, the scope is probably still fuzzy.

**Sketch the data model first.** What new tables, fields, or relationships does this need? Even a rough ERD or bullet list on paper saves hours later. Changes to the data model ripple through everything — migrations, serializers, frontend types, API calls — so it's the cheapest thing to change at the whiteboard stage and the most expensive once code is written.

**Identify the API surface.** What endpoints will the frontend need? GET, POST, PATCH, DELETE? What shape does the response take? Write them down before implementing anything.

**Talk to a teammate early.** A 15-minute conversation before writing any code is worth more than a day of rework later. If the feature touches an area you're less familiar with (auth, permissions, a complex existing module), the person who owns that area should weigh in on approach.

**When in doubt, write a short design doc.** Not a formal document — just a paragraph or two in the issue covering the proposed approach, alternatives considered, and any trade-offs. Reviewers at PR time will thank you.

---

## 2. UI prototyping

For any feature with a UI component, build a prototype **before** writing the backend. This sounds backwards, but it catches design issues while they're still cheap to fix.

**Build a static prototype first.** Hard-code the data, skip the API integration, and get the UI looking roughly right. This can be a React component with dummy data, or even just a Figma-style mockup in code. The goal is to answer the question: "Is this actually what we want?" before spending time on backend plumbing.

!!! tip "Get eyes on it early"
    Share the prototype in Slack or in a draft PR and ask for feedback **before** wiring up the backend. It is dramatically cheaper to move a button than to refactor three API endpoints because the UI changed shape.

**Match the styling of the most recently deployed feature.** Consistency matters more than cleverness. When starting a new UI, open the most recently merged feature and model yours after it — same spacing, same component patterns, same color choices. This keeps the product feeling coherent instead of like a patchwork.

Specifically:

- Use the same spacing scale, button sizes, and form patterns as recent work
- Pull from `shared/components/ui/` wherever possible instead of creating one-off components
- If you find yourself copy-pasting styles from an existing page, consider whether the thing being copied belongs in `shared/` instead

**Consult teammates on visual decisions.** Before introducing a new color, icon style, or layout pattern, ask in the team channel or loop in whoever built the most recent similar feature. Visual decisions are hard to unwind once they ship — we'd rather catch "wait, that doesn't match the rest of the app" at prototype stage than in PR review.

**Iterate on the prototype until everyone's happy.** Only then do you start the real backend work.

---

## 3. Backend planning

With the UI shape clear, you now know exactly what data the frontend needs. Plan the backend around that.

The rough order:

**1. Models first.** See [Models](../backend/models.md) for conventions. Sketch the model, all fields, relationships, and `Meta` options. Think about:

- What indexes will queries need?
- What `on_delete` behavior makes sense for foreign keys?
- Which fields are nullable? Which have defaults?
- Do you need constraints (`UniqueConstraint`, `CheckConstraint`)?

**2. Generate and inspect migrations.** Run `makemigrations` and **read the generated file** before running it. Migrations are code — bad migrations can lock tables, drop data, or make downtime inevitable later. If the generated migration does something unexpected, pause and figure out why.

!!! warning "Always commit migrations"
    Never leave generated migrations uncommitted. Your PR must include them, even if they feel "throwaway" during development.

**3. Serializers next.** See [Serializers](../backend/serializers.md). Typically you'll need:

- A **list serializer** (lean — just what shows on the list view)
- A **detail serializer** (full object)
- A **create/update serializer** (only the writable fields)

Don't reuse one serializer for everything. Separate concerns keep payloads small and validation targeted.

**4. Views and URLs.** See [Views](../backend/views.md) and [URLs](../backend/urls.md). For a standard CRUD resource, use a `ModelViewSet` with a router. For anything custom, pick the simplest primitive that works:

- Full CRUD → `ModelViewSet`
- Mostly list/create → `ListCreateAPIView` or similar
- One-off endpoint → `@api_view` function

**5. Permissions and filtering.** Filter in `get_queryset()` so users can only see their own data. Set `permission_classes` explicitly. Don't rely on the frontend to hide buttons — always enforce on the backend.

**6. Tests where they matter most.** At minimum: tests for permissions (can the wrong user access this?) and for any non-trivial business logic. Don't chase 100% coverage on trivial getters.

---

## 4. Frontend planning

With the API shape defined, the frontend work is mostly plumbing.

The rough order:

**1. Types first.** See [Project Structure → Types](../frontend/project-structure.md#types-folder). Write interfaces that match the serializer output exactly — use snake_case to match DRF, and keep the types in `types/index.ts`. This is the contract between frontend and backend.

**2. API file.** See [API Files](../frontend/api.md). One method per endpoint, all typed, no business logic. Just request → response.

**3. Hooks.** See [Hooks](../frontend/hooks.md). A data-fetching hook for loading the resource, an action hook for mutations. Keep components ignorant of how data gets fetched.

**4. Components last.** See [Components](../frontend/components.md). With types, API, and hooks in place, components are mostly JSX plus calls to your hooks. If a component is getting complicated, check whether logic should move up into a hook or out into `utils/`.

---

## 5. Building incrementally

Don't write the whole feature before running anything. Build in thin vertical slices.

**A good rhythm:**

1. Ship one endpoint + one screen that shows the data (read-only first)
2. Verify it works end-to-end in the browser
3. Add write actions one at a time
4. Add edge cases, error states, loading states last

**Commit often.** Small commits are easier to review and easier to revert. Each commit should leave the codebase in a working state.

**Keep a draft PR open from early.** This gives the team visibility into what you're working on and surfaces problems sooner. See [Draft PRs](pull-requests.md#draft-prs).

---

## 6. PR and review

Once the feature is working end-to-end and you're happy with it, convert the draft PR to ready-for-review.

**Before marking ready, self-review the diff.** Open the Files Changed tab and read through your own PR as if you were the reviewer. You'll catch things: leftover `console.log`s, commented-out code, a test you meant to write.

**Write a good PR description.** See [Pull Requests → Titles & Descriptions](pull-requests.md#titles-descriptions). For a larger feature, the description should cover:

- What this adds and why
- Any unusual decisions or trade-offs
- How to test it locally (setup steps, which URL to visit, which user to log in as)
- Screenshots or a short screen recording of the UI
- Link to the ticket/issue

**Tag the right reviewers.** One or two people, ideally including someone who knows the area well. Don't tag the whole team.

**Respond to feedback promptly.** Try to turn around PR comments within a day. If you disagree with a comment, explain your reasoning — don't just dismiss it.

---

## 7. Deploy

For now, deploys happen manually after merge. In the future, CI/CD will handle this automatically — see [CI/CD](ci-cd.md).

**Before merging, verify:**

- All CI checks pass
- Required approvals received
- Migrations are committed (if any)
- No secrets accidentally committed

**After deploy, verify in production:**

- The feature actually works with real data (not just your local test fixtures)
- No errors showing up in logs
- Existing functionality still works (smoke test the most common flows)

**If something breaks, revert first and investigate later.** It's almost always faster to roll back a bad deploy than to hotfix forward under pressure.

---

## A realistic example

Here's roughly how building a feature like "Locate management" would go, to make the above concrete:

| Phase | What happens                                                                   |
|-------|--------------------------------------------------------------------------------|
| 1     | **Ticket filed:** Users need to request underground utility locates.           |
| 2     | **Design doc:** Locate has a job, location, expiration, and status lifecycle. Data model sketched in the issue. |
| 3     | **UI prototype:** List view + detail view + create form, all with hardcoded data. Shared in Slack. |
| 4     | **Team review:** Someone points out the status badges don't match the existing work orders page. Adjust. |
| 5     | **Model + migration:** Written, reviewed, committed. |
| 6     | **Serializers:** Three of them — `LocateListSerializer`, `LocateDetailSerializer`, `LocateCreateSerializer`. |
| 7     | **ViewSet + URL routing:** `ModelViewSet` with `get_queryset()` filtering by company. |
| 8     | **Types + API file:** `locatesApi.list()`, `locatesApi.create()`, etc. |
| 9     | **Hooks:** `useLocates(jobId)` for data, `useLocateActions()` for mutations. |
| 10    | **Wire up UI:** Replace the hardcoded data in the prototype with real hook calls. |
| 11    | **Test end-to-end:** Create a locate, see it in the list, verify expiration logic. |
| 12    | **PR:** Draft → ready for review → approved → merged. |
| 13    | **Deploy + verify:** Smoke test on production with real data. |

Each phase should feel like it builds on the previous one. If you're stuck on step 5 and keep having to go back and change things in step 2, that's a signal the earlier planning wasn't thorough enough — worth pausing to rethink rather than pushing through.

---

## Related reading

- [Pull Requests](pull-requests.md) — how to write and review PRs
- [Support Email Process](support-email.md) — how we handle the shared inbox
- [CI/CD](ci-cd.md) — planned automation (not yet built)
- [General Principles](../general-principles.md) — language-agnostic practices
