# Frontend

React + TypeScript patterns and conventions.

## Pages in this section

- **[Project Structure](project-structure.md)** — how apps are organized under `src/`.
- **[Components](components.md)** — file structure, props, keeping components focused.
- **[Hooks](hooks.md)** — data-fetching, actions, return shapes.
- **[API Files](api.md)** — structure, naming, rules.

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
