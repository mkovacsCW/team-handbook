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
