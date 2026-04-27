# Alerts

Reference for the custom alert dialog — when to use it, how to call it, and the options it supports.

For the high-level "alerts vs. toasts" question, see [Components → User feedback](components.md#user-feedback-alerts-and-toasts).

## Why we have a custom alert dialog

The browser's `window.alert()` and `window.confirm()` work, but:

- They look out of place against the rest of the app — no theming, no styling, no control over typography.
- They block the entire window in a way that feels heavy and disruptive.
- Behaviour varies between browsers (and on some, they don't respect OS dark mode).
- They can't show different visual treatments for info vs. warning vs. error vs. success.
- They're synchronous, which makes them awkward to compose with `async`/`await` flows.

The custom `AlertDialog` is a styled, animated, portal-rendered modal that fixes all of the above. It's promise-based, so you can `await` the user's choice and branch on it cleanly.

**Use it any time you would have written `alert(...)` or `confirm(...)`.**

---

## When to use an alert

Alerts interrupt the user. Reach for one when:

- The user has to make a decision before continuing ("Are you sure you want to delete this?")
- The message is important enough that the user *must* see it before doing anything else (destructive actions, irreversible changes)
- You need a yes/no answer back from the user

If the user doesn't have to act on the message, use a [toast](toasts.md) instead. Alerts that don't ask anything of the user feel like the app is yelling.

---

## Quick Start

### 1. Import the hook

```tsx
import { useAlert } from '@/shared/components/ui/AlertDialog';
```

### 2. Call the hook in your component

```tsx
const MyComponent = () => {
  const { showAlert, AlertComponent } = useAlert();

  // ... rest of component

  return (
    <>
      {/* your UI */}
      {AlertComponent}
    </>
  );
};
```

The hook returns:

- `showAlert(config)` — function that displays the alert and returns a promise resolving to `true` (confirmed) or `false` (cancelled)
- `AlertComponent` — JSX you must render somewhere in your component tree for the alert to appear

### 3. Show an alert

```tsx
const handleDelete = async () => {
  const confirmed = await showAlert({
    type: 'warning',
    title: 'Delete this locate?',
    message: 'This action can\'t be undone.',
    confirmText: 'Delete',
    cancelText: 'Cancel',
  });

  if (confirmed) {
    await deleteLocate(id);
  }
};
```

---

## Options Reference

| Option         | Type                                            | Default     | Description                                                |
|----------------|-------------------------------------------------|-------------|------------------------------------------------------------|
| `message`      | `string`                                        | *required*  | Body text of the alert                                     |
| `title`        | `string`                                        | `undefined` | Optional heading shown above the message                   |
| `type`         | `'info' \| 'warning' \| 'error' \| 'success'`   | `'info'`    | Visual treatment — sets icon and accent colour             |
| `confirmText`  | `string`                                        | `'OK'`      | Label for the primary action button                        |
| `cancelText`   | `string`                                        | `'Cancel'`  | Label for the secondary action button                      |
| `showCancel`   | `boolean`                                       | `true`      | Set `false` for an info-only alert with no cancel option   |

---

## Common Patterns

### Confirming a destructive action

```tsx
const confirmed = await showAlert({
  type: 'warning',
  title: 'Terminate this locate?',
  message: 'The locate will be marked terminated and removed from the active list.',
  confirmText: 'Terminate',
  cancelText: 'Keep it',
});

if (confirmed) {
  await terminateLocate(locateId);
}
```

### Surfacing an error that needs acknowledgement

```tsx
await showAlert({
  type: 'error',
  title: 'Sync failed',
  message: 'We couldn\'t reach the server. Check your connection and try again.',
  confirmText: 'OK',
  showCancel: false,
});
```

### Confirming success when the user should explicitly acknowledge it

Most success feedback should be a [toast](toasts.md), not an alert. Only use a success alert when the user genuinely needs to read and dismiss it before continuing — e.g. a confirmation number they need to write down.

```tsx
await showAlert({
  type: 'success',
  title: 'Locate submitted',
  message: `Your confirmation number is ${confirmationNumber}.`,
  confirmText: 'Got it',
  showCancel: false,
});
```

---

## Behaviour

- **Backdrop click** doesn't dismiss the alert — instead the dialog pulses to draw attention back to the action buttons. This is intentional: alerts demand a decision.
- **Escape key** has the same pulse behaviour. The user must explicitly choose confirm or cancel.
- **Focus** lands on the confirm button when the alert opens, so a quick `Enter` confirms.
- **Animations** for enter / leave / pulse are built in and respect the same motion principles described in [Components → Animations](components.md#animations-and-transitions).

---

## Don'ts

- **Don't use an alert when the user has nothing to decide.** "Saved successfully!" should be a toast, not an alert.
- **Don't use an alert as a substitute for a real form or modal.** If you need more than a confirm/cancel decision, build a proper modal.
- **Don't stack alerts.** If you need to ask multiple questions, use a single dialog or step the user through them sequentially.
- **Don't fall back to `window.alert` or `window.confirm` "just for now"** — they look broken against the rest of the app and tend to never get replaced.