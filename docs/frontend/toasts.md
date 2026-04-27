# Toasts

Reference for the toast notification system — when to use them, how to call them, and how to handle the loading → success/error pattern around server requests.

For the high-level "alerts vs. toasts" question, see [Components → User feedback](components.md#user-feedback-alerts-and-toasts).

## What toasts are for

Toasts are the standard way we tell the user that something happened — a save succeeded, a sync failed, a request is in flight. They appear briefly, don't block the UI, and don't demand any action from the user.

**Use a toast whenever the app does something on the user's behalf and they should know about it but don't need to decide anything.** That covers basically all server-side mutations: create, update, delete, sync, submit.

If the user needs to make a decision, use an [alert](alerts.md) instead.

---

## The loading → success / error pattern

This is the pattern we want around any server request that mutates data. It's worth getting right because it's what makes the app feel responsive instead of feeling like it's silently doing something (or, worse, broken).

The flow:

1. User triggers an action (clicks "Save", submits a form, etc.)
2. Show a **loading toast** immediately — confirms the action was registered.
3. When the request resolves:
   - On success: dismiss the loading toast, show a **success toast**.
   - On error: dismiss the loading toast, show an **error toast** with a useful message.

```tsx
const { showToast, dismissToast } = useToast();

const handleSubmit = async () => {
  const loadingId = showToast({
    type: 'loading',
    message: 'Saving locate…',
  });

  try {
    await locatesApi.submit(formData);
    dismissToast(loadingId);
    showToast({
      type: 'success',
      message: 'Locate saved',
    });
  } catch (err) {
    dismissToast(loadingId);
    showToast({
      type: 'error',
      message: 'Couldn\'t save locate — please try again',
    });
  }
};
```

A few rules for this pattern:

- **Loading toasts don't auto-dismiss.** They stay until you call `dismissToast(id)`. Don't forget to dismiss them in *both* the success and error branches — including in any early returns.
- **Error messages should be useful.** "Something went wrong" is almost as bad as no message at all. Tell the user what failed and, where possible, what they can do about it.
- **For very fast requests (< 200ms), you can skip the loading toast** and just show success directly. A loading toast that flashes on screen for 100ms is just visual noise.

---

## Quick Start

### 1. Import the hook

```tsx
import { useToast } from '@/shared/components/ui/ToastNotification';
```

### 2. Call the hook in your component

```tsx
const MyComponent = () => {
  const { showToast, dismissToast, ToastContainer } = useToast();

  // ... rest of component

  return (
    <>
      {/* your UI */}
      {ToastContainer}
    </>
  );
};
```

The hook returns:

- `showToast(config)` — displays a toast and returns its `id` (string), which you can use to dismiss it manually
- `dismissToast(id)` — removes a toast immediately
- `ToastContainer` — JSX you must render somewhere in your component tree for toasts to appear

### 3. Show a toast

```tsx
showToast({
  type: 'success',
  message: 'Changes saved',
});
```

---

## Options Reference

| Option      | Type                                          | Default     | Description                                                       |
|-------------|-----------------------------------------------|-------------|-------------------------------------------------------------------|
| `message`   | `string`                                      | *required*  | Text shown in the toast                                           |
| `type`      | `'success' \| 'error' \| 'info' \| 'loading'` | *required*  | Sets the icon, colour, and auto-dismiss behaviour                 |
| `duration`  | `number` (ms)                                 | `3000`      | How long until the toast auto-dismisses. Ignored for `loading`.   |

**Behaviour by type:**

| Type      | Auto-dismisses? | Use for                                                 |
|-----------|-----------------|---------------------------------------------------------|
| `success` | Yes             | Confirming a successful action                          |
| `error`   | Yes             | Reporting a failure                                     |
| `info`    | Yes             | Neutral status updates ("Sync started", "Copied to clipboard") |
| `loading` | **No** — must dismiss manually | In-flight requests                       |

---

## Common Patterns

### Standard mutation flow

The bread-and-butter pattern. Wrap any server mutation:

```tsx
const handleDelete = async (id: number) => {
  const loadingId = showToast({ type: 'loading', message: 'Deleting…' });

  try {
    await api.delete(id);
    dismissToast(loadingId);
    showToast({ type: 'success', message: 'Deleted' });
  } catch {
    dismissToast(loadingId);
    showToast({ type: 'error', message: 'Failed to delete — please try again' });
  }
};
```

### Quick confirmation without a loading toast

For very fast requests where the loading state would just flash:

```tsx
try {
  await api.markAsRead(id);
  showToast({ type: 'success', message: 'Marked as read' });
} catch {
  showToast({ type: 'error', message: 'Couldn\'t update — please try again' });
}
```

### Letting a toast persist longer

If the message has more content the user might want time to read:

```tsx
showToast({
  type: 'info',
  message: 'Sync started — this can take up to a minute',
  duration: 6000,
});
```

### Letting a toast persist indefinitely

Set `duration: 0` to keep a non-loading toast on screen until manually dismissed. Use sparingly — most messages should auto-dismiss.

---

## Don'ts

- **Don't show a toast for every keystroke or trivial UI change.** Toasts are for things that crossed a network boundary or had real consequences. Local UI feedback (a button toggling, a checkbox flipping) doesn't need a toast.
- **Don't use a toast for something the user must read.** If they have to acknowledge it, it's an [alert](alerts.md), not a toast.
- **Don't stack toasts when one would do.** If five things just succeeded, show "5 items saved" — not five separate success toasts.
- **Don't forget to dismiss loading toasts.** Every code path that started a loading toast must end with either `dismissToast` (success or error) or the toast will hang on screen forever.
- **Don't hide errors behind generic messages.** "Something went wrong" tells the user nothing. If you have an error message from the server, surface a useful version of it.

---

## Why this matters

The toast pattern is one of the highest-leverage UI investments in the app. A user submitting a form and getting *no feedback* is the single most common reason something feels broken — even when it worked. Conversely, a smooth loading-toast → success-toast flow makes even slow requests feel intentional and responsive. It's worth the few extra lines of code on every mutation.