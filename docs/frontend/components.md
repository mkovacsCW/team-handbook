# Components

## File Structure

Follow a consistent internal structure for every component file:

```tsx
// 1. React/library imports
import { FC, useState, useEffect, useMemo } from 'react';
import { Link } from 'react-router-dom';

// 2. Internal/shared imports
import { AlertDialog } from '@/shared/components/ui/AlertDialog';
import { MapPinIcon, SyncIcon } from '@/shared/components/icons';

// 3. Local imports (hooks, types, utils)
import { useLocates } from '../hooks/useLocates';
import { LocateListItem } from '../types';
import { getStatusBadgeClass, formatDateLocal } from '../utils';

// 4. Styles
import '../styles/locates-card.css';

// 5. Props interface
interface LocatesCardProps {
  jobId: number;
  date: string;
  userRole: string;
}

// 6. Component
const LocatesCard: FC<LocatesCardProps> = ({ jobId, date, userRole }) => {
  // State
  // Hooks
  // Effects
  // Handlers
  // Computed/memos
  // Render
};

// 7. Export
export default LocatesCard;
```

## Props Typing

Always define a typed interface for props. Avoid `any` — if the shape is defined elsewhere, import it:

```tsx
// Bad
interface LocatesCardProps {
  jobData: any;
  onSubmit: Function;
}

// Good
interface LocatesCardProps {
  jobData: Job;
  onSubmit: (locateId: number) => void;
}
```

For callback props, use the `on[Event]` prefix. For internal handlers, use `handle[Action]`:

```tsx
// Props
interface LocatesCardProps {
  onRelocate: (locate: LocateListItem) => void;
  onTerminate: (locate: LocateListItem) => void;
}

// Inside the component
const handleRelocate = (locate: LocateListItem) => {
  // validation, prep, etc.
  onRelocate(locate);
};
```

## Keeping Components Focused

!!! warning "Signs a component is doing too much"
    - More than ~1000 lines
    - 8+ state variables
    - Multiple unrelated `useEffect` blocks
    - Duplicated logic across components (sort icons, status badges, date formatting)
    - Business logic mixed into the render (date math, status calculations)
    - Inline SVGs repeated across files

### How to fix it

**Move reusable display logic into `utils/`:**

```tsx title="utils/index.ts"
export const getStatusBadgeClass = (status: string): string => {
  switch (status) {
    case 'Active': return 'active';
    case 'Expiring Soon': return 'expiring';
    case 'Expired': return 'expired';
    case 'Terminated': return 'terminated';
    case 'Requested': return 'requested';
    default: return '';
  }
};

export const formatDateLocal = (dateString: string | null): string => {
  if (!dateString) return '—';
  const [year, month, day] = dateString.split('-').map(Number);
  return new Date(year, month - 1, day).toLocaleDateString('en-US', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
  });
};
```

**Move complex stateful logic into custom hooks:**

```tsx
// Bad — business logic in the component
const LocatesCard = () => {
  const [sortConfig, setSortConfig] = useState({ key: 'expiration_date', direction: 'ascending' });

  const handleSort = (key: string) => { /* ... */ };
  const sortedLocates = useMemo(() => { /* 20 lines of sort logic */ }, [locates, sortConfig]);
  const filteredLocates = useMemo(() => { /* filtering logic */ }, [sortedLocates, searchQuery]);
};

// Good — extracted to a hook
const LocatesCard = () => {
  const { sortedItems, handleSort, getSortDirection } = useSort(locates, {
    key: 'expiration_date',
    direction: 'ascending',
  });
  const filteredLocates = useFilter(sortedItems, searchQuery, [
    'location_name',
    'locate_number',
    'status',
  ]);
};
```

**Use shared icon components instead of inline SVGs:**

```tsx
// Bad — inline SVG repeated across files
<svg width="20" height="20" viewBox="0 0 24 24" fill="none">
  <path d="M12 2C8.13 2 5 5.13 5 9C5 ..." fill="currentColor" />
</svg>

// Good — reusable icon component
import { MapPinIcon } from '@/shared/components/icons';
<MapPinIcon size={20} />
```

## What Belongs in a Component

| Belongs in component          | Move somewhere else                            |
|-------------------------------|------------------------------------------------|
| State directly tied to UI     | Data fetching logic → hooks                    |
| Event handlers (thin)         | Business logic / validation → hooks or utils   |
| Conditional rendering         | Repeated display helpers → utils               |
| Layout and JSX                | Reusable UI pieces → shared components         |
| Component-specific memos      | Inline SVGs → icon components or icon library  |

## UI Quality

The look and feel of a component is part of its job, not a finishing touch. Users form their judgment of the whole product from the small UI cues — how things load, how they move, how responsive a click feels. A technically correct component that snaps awkwardly between states still feels broken, and "feels broken" is what users remember.

Treat the items below as part of "done", not as polish you'll add later.

### Loading states

Every component that depends on async data needs a deliberate loading state. Don't let the UI show empty space, a flicker of "no results", or stale data while a request is in flight.

In general, any data-driven view should have a clear answer for four situations: it's loading, it errored, it loaded successfully but there's nothing to show, and it loaded with data. If you only handle the first and last, the other two will surface as bugs. Loading and empty states should look visually distinct from each other — "no data yet" should feel intentional, not like a screen that's still working.

For actions (submits, deletes, anything that fires a request), the trigger should reflect that something is happening and should not be re-triggerable until the request resolves. Otherwise users double-click and you get duplicate submissions.

### Animations and transitions

Use motion to **explain change**, not to decorate. A good rule of thumb: if the user's mental model just shifted — something appeared, disappeared, expanded, sorted, moved — a short transition helps them follow what happened. If nothing changed, don't animate.

Places where motion earns its keep: modals and popovers entering and leaving, dropdowns opening, collapsible sections expanding, list reorders after sort or filter changes, route transitions, and hover/focus feedback on interactive elements.

Keep it tasteful. 150–300ms covers almost everything; anything over 400ms starts to feel slow. Use `ease-out` for things entering and `ease-in` for things leaving. And always respect `prefers-reduced-motion` — when the user has reduced motion turned on at the OS level, animations should collapse to instant transitions.

### User feedback: alerts and toasts

Two specific UI primitives we use across the app to communicate with the user. Both have in-house implementations, and **you should use those rather than rolling your own or falling back to browser defaults.**

- **Alerts** — for situations where the user has to make a decision before continuing ("Are you sure you want to delete this?"). Use the custom alert dialog instead of `window.alert` / `window.confirm`. See **[Alerts](alerts.md)**.
- **Toasts** — for transient feedback after the app does something on the user's behalf (saved, deleted, sync failed). The standard pattern is loading → success/error around any server mutation. See **[Toasts](toasts.md)**.

Quick rule of thumb for picking between them: if the user needs to *act* on the message, it's an alert. If they just need to *know* something happened, it's a toast. If you're tempted to use an alert for something the user doesn't have to act on, use a toast instead — alerts that don't ask anything of the user feel like the app is yelling.

### Why this matters

A user can't see your reducer logic or your typed props. What they can see is whether the page felt fast, whether the button responded, whether the modal slid in or jumped in. **Good UI heavily determines how users interpret the entire product** — including the parts you spent the most engineering effort on. Components that nail loading states and have considered motion feel professional and trustworthy. Ones that don't feel buggy, even when they aren't.

When in doubt, look at the most recently shipped feature and match its loading and motion patterns (see [Systems Development → UI Prototyping](../workflow/systems-development.md#2-ui-prototyping)).

## Smooth Scrolling

For long lists, scroll-heavy modals, or any view where native scrolling feels stuttery — particularly on trackpads or with heavy DOM content — we have a `useSmoothScroll` hook that takes over scroll handling on a container and applies friction-based easing.

**When to use it:**

- Long content inside a modal where native scrolling feels jumpy
- Large lists or tables where the user does a lot of scrolling
- Any view where the perceived smoothness of scrolling materially affects how the page feels

**When *not* to use it:**

- Short content that fits in the viewport
- Pages where native scroll already feels fine — don't fix what isn't broken
- Anywhere the user has `prefers-reduced-motion: reduce` set (the hook supports this via its `disabled` option — wire it up)

The hook is opinionated about its DOM and CSS setup (sole scroll owner, `position: absolute; inset: 0` on the inner container, no competing `overflow` or `scroll-behavior` in the chain). Read the full guide before integrating: **[Smooth Scrolling](smooth-scroll.md)**.