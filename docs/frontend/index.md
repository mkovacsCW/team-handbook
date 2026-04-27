# Frontend

React + TypeScript patterns and conventions.

!!! warning "Heads up: not every file follows these conventions"
    Parts of the codebase predate this handbook, and some files still use styling and components from the **Metronic** front-end package we inherited. The conventions on the following pages describe **where we're going**, not always where every file currently is.

    **Our direction is to remove Metronic entirely.** That means:

    - Don't add new Metronic-styled components, layouts, or theme classes — build replacements yourself using the patterns in this handbook and the components in `shared/components/ui/`.
    - When you touch a Metronic-based file for unrelated work, you don't need to rewrite it in the same PR (see [Clean as You Go](../workflow/general-principles.md#11-clean-as-you-go) — keep cleanup proportional). But prefer migrating it over patching it if the change is non-trivial.
    - Over time, deprecate and delete Metronic-related files as they're replaced.

    **Found a file that looks unused or dead?** Don't just delete it — ping **Marton** or **Logan** first. They'll know whether it's actually safe to remove or whether something obscure still depends on it.

## Pages in this section

- **[Project Structure](project-structure.md)** — how apps are organized under `src/`.
- **[Components](components.md)** — file structure, props, keeping components focused.
- **[Hooks](hooks.md)** — data-fetching, actions, return shapes.
- **[API Files](api.md)** — structure, naming, rules.
- **[Alerts](alerts.md)** — the custom alert dialog and when to use it over `window.alert`.
- **[Toasts](toasts.md)** — toast notifications and the loading → success/error pattern.
- **[Smooth Scrolling](smooth-scroll.md)** — the `useSmoothScroll` hook and integration patterns.

## Quick reference

| Type       | Convention            | Example                               |
|------------|-----------------------|---------------------------------------|
| Folders    | kebab-case            | `user-profile/`                       |
| Components | PascalCase            | `UserCard.tsx`                        |
| Hooks      | camelCase, `use` prefix | `useLocates.ts`                     |
| API        | camelCase, `Api` suffix | `locatesApi.ts`                     |
| Types      | PascalCase            | `LocateDetail`                        |
| Styles     | kebab-case            | `user-card.scss`                      |
| Constants  | SCREAMING_SNAKE_CASE  | `export const APP_NAME = 'MyApp';`    |