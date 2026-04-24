# Project Structure

All app subfolders should follow the same structure: `components/`, `hooks/`, `api/`, `styles/`, `types/`, `constants/`, `index.ts`. Folder names use kebab-case.

## Example structure

```
Front-End/src/
├── apps/
│   ├── auth/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── styles/
│   │   ├── types/
│   │   ├── constants/
│   │   └── index.ts
│   ├── dashboard/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── styles/
│   │   ├── types/
│   │   └── index.ts
│   └── orders/
│       ├── api/
│       ├── components/
│       ├── hooks/
│       ├── styles/
│       ├── types/
│       └── index.ts
├── shared/
│   ├── api/
│   ├── components/
│   │   ├── layout/
│   │   └── ui/
│   ├── contexts/
│   ├── hooks/
│   ├── routes/
│   ├── types/
│   ├── utils/
│   └── styles/
├── config/
└── App.tsx
```

## Components folder

If there are fewer than 10 component files, keep a flat structure. All files should be PascalCase and the filename should make it obvious what type of component it is without opening the file (`WorkerMetricsStatsCard.tsx`, not `StatsWidget2.tsx`).

```
apps/dashboard/
└── components/
    ├── StatsCard.tsx
    ├── FilterModal.tsx
    └── AnalyticsPage.tsx
```

If there are more than 10 components, split them into subfolders by type:

```
apps/dashboard/
└── components/
    ├── pages/
    ├── cards/
    ├── modals/
    └── layout/
```

## Hooks folder

camelCase with `use` prefix. Should follow the name of the app or feature:

- `use[App].ts` — e.g., `useLocates.ts`, `useAuth.ts`
- `use[App][Action].ts` — e.g., `useLocatesSync.ts`, `useAuthLogin.ts`

Use `.ts`, not `.tsx` — hook files don't contain JSX.

```
hooks/
├── useLocates.ts
├── useLocatesSync.ts
└── useLocatesFilter.ts
```

## API folder

camelCase ending with `Api`. Should follow the name of the app: `[app]Api.ts`. Use `.ts`, not `.tsx`.

```
api/
└── locatesApi.ts
```

## Types folder

All types in a single `index.ts` file within the `types/` folder. Use PascalCase for interface/type names. Use snake_case for interface properties to match Django REST Framework responses.

```typescript title="types/index.ts"
export interface Locate {
  id: number;
  locate_number: string;
  location_name: string;
  request_date: string;
  expiration_date: string | null;
}

export interface LocateDetail extends Locate {
  project_number: string;
  project_name: string;
  requested_by: { id: number; full_name: string } | null;
  created_at: string;
  updated_at: string;
}

// Sync-related
export interface SyncStatus {
  last_sync: string | null;
  can_sync: boolean;
  minutes_remaining: number;
}
```

## Constants folder

All constants in `index.ts` within `constants/`. SCREAMING_SNAKE_CASE.

```typescript
export const PROPERTY_TYPE_OPTIONS = [
  'Public',
  'Private',
  'Both',
] as const;

export const INVALID_LOCATION_NAME_CHARS = /['"#./\\:,<>|?*]/;
```

## Styles folder

kebab-case. Should match the component name in lowercase: `[component-name].scss`.

```
styles/
├── locate-card.scss
├── locate-table.scss
└── locates-page.scss
```

## Index files

Index files act as the "public interface" for a folder. They let you control what gets exported and simplify imports.

=== "Without index files"

    Imports must reference the exact file path:

    ```typescript
    import { LoginForm } from '@/apps/auth/components/LoginForm';
    import { RegisterForm } from '@/apps/auth/components/RegisterForm';
    import { useAuth } from '@/apps/auth/hooks/useAuth';
    import type { User } from '@/apps/auth/types';
    ```

=== "With index files"

    ```typescript title="apps/auth/index.ts"
    export { LoginForm } from './components/LoginForm';
    export { RegisterForm } from './components/RegisterForm';
    export { useAuth } from './hooks/useAuth';
    export * from './types';
    export * from './api/authApi';
    ```

    ```typescript
    import { LoginForm, RegisterForm, useAuth } from '@/apps/auth';
    import type { User, AuthResponse } from '@/apps/auth';
    ```

**Tips:**

- Only export what other parts of the app need — internal helpers can stay private
- Use `export *` for types and api, explicit exports for components
- If you rename or move a file internally, only update the index file

## Shared folder

Contains code used across multiple apps or globally. If something is only used by one app, keep it in that app's folder instead.

```
shared/
├── api/              # axiosInstance, base config
├── components/
│   ├── layout/       # Navbar, Footer, AppLayout
│   └── ui/           # Button, Modal, Input
├── contexts/         # UserContext, ThemeContext
├── hooks/            # useDebounce, useFetch
├── routes/           # PrivateRoutes
├── types/            # shared types, generics
├── utils/            # formatDate, parseError
└── styles/           # global styles, variables
```

**When to promote something to `shared/`:**

- Component is used by 2+ apps (e.g., `Button`, `Modal`)
- Context is app-wide (e.g., `UserContext`)
- Hook is generic and reusable (e.g., `useDebounce`)
- Route guards (e.g., `PrivateRoutes`)
- Global layout components (e.g., `Navbar`, `Footer`)

## Config folder

Application-wide configuration, constants, and environment-related settings.

```
config/
├── constants.ts      # app-wide constants
├── env.ts            # environment variables
└── routes.ts         # route path definitions
```

```typescript title="config/constants.ts"
export const APP_NAME = 'MyApp';
export const PAGINATION_LIMIT = 20;
export const DATE_FORMAT = 'YYYY-MM-DD';
```

```typescript title="config/routes.ts"
export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  DASHBOARD: '/dashboard',
  LOCATES: '/locates',
  LOCATE_DETAIL: (id: number) => `/locates/${id}`,
} as const;
```

```typescript title="config/env.ts"
export const ENV = {
  API_URL: import.meta.env.VITE_API_URL,
  DEBUG: import.meta.env.VITE_DEBUG === 'true',
} as const;
```
