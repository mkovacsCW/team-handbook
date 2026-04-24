# Hooks

## Data-Fetching Hooks

Follow this standard pattern for hooks that fetch data on mount or when a dependency changes:

```tsx
interface UseLocatesResult {
  locates: LocateListItem[];
  isLoading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

export const useLocates = (jobId: number | null): UseLocatesResult => {
  const [locates, setLocates] = useState<LocateListItem[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchLocates = useCallback(async () => {
    if (!jobId) {
      setLocates([]);
      return;
    }

    setIsLoading(true);
    setError(null);

    try {
      const data = await locatesApi.list(jobId);
      setLocates(data);
    } catch (err) {
      console.error('Error fetching locates:', err);
      setError('Failed to load locates');
      setLocates([]);
    } finally {
      setIsLoading(false);
    }
  }, [jobId]);

  useEffect(() => {
    fetchLocates();
  }, [fetchLocates]);

  return { locates, isLoading, error, refetch: fetchLocates };
};
```

**Key points:**

- Always return `isLoading`, `error`, and a `refetch` function
- Guard against null/undefined params early
- Reset state on error so stale data doesn't linger
- Use `useCallback` for the fetch function so it can be passed as a dependency

## Action Hooks

For hooks that perform mutations (create, update, delete), group related actions and track loading state per action:

```tsx
interface UseLocateActionsResult {
  submitLocate: (formData: FormData) => Promise<{ success: boolean; locateId?: number }>;
  relocateLocate: (locateId: number) => Promise<boolean>;
  terminateLocate: (locateId: number) => Promise<boolean>;
  isSubmitting: boolean;
  isRelocating: boolean;
  isTerminating: boolean;
}

export const useLocateActions = (): UseLocateActionsResult => {
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isRelocating, setIsRelocating] = useState(false);
  const [isTerminating, setIsTerminating] = useState(false);

  const submitLocate = useCallback(async (formData: FormData) => {
    setIsSubmitting(true);
    try {
      const response = await locatesApi.submit(formData);
      return { success: response.success, locateId: response.locateId };
    } catch (err) {
      console.error('Error submitting locate:', err);
      return { success: false };
    } finally {
      setIsSubmitting(false);
    }
  }, []);

  // ... other actions follow the same pattern

  return {
    submitLocate,
    relocateLocate,
    terminateLocate,
    isSubmitting,
    isRelocating,
    isTerminating,
  };
};
```

**Key points:**

- Each action gets its own `is[Action]` loading flag
- Actions return a result (success/failure + data) rather than throwing
- Wrap every action in `useCallback` with an empty dependency array
- Keep the hook focused — if actions are unrelated, use separate hooks

## Return Shapes

Always return an **object**, not a tuple. Objects are easier to destructure selectively and more readable:

```tsx
// Good — object return
const { locates, isLoading, refetch } = useLocates(jobId);

// Avoid — tuple return (fine for simple 2-value hooks, but doesn't scale)
const [locates, isLoading, error, refetch] = useLocates(jobId);
```

## When to Extract a Hook

- The same `useState` + `useEffect` pattern appears in 2+ components
- A component has 3+ state variables that work together (e.g., sort key + direction + sorted data)
- Complex `useEffect` logic that's hard to follow inline
- Any reusable stateful behavior (debounce, polling, pagination)
