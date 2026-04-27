# Smooth Scrolling

Reference for the `useSmoothScroll` hook — when to use it, how to integrate it, and the CSS rules that make it work.

For the high-level "should I use this?" question, see [Components → Smooth Scrolling](components.md#smooth-scrolling). This page is the practical integration guide.

## How It Works

The hook intercepts native `wheel` and `touch` events on a container, converts them into velocity, and runs a `requestAnimationFrame` loop that applies friction decay and lerp-based interpolation to `scrollTop`. A companion `<Scrollbar />` component renders a custom scrollbar that tracks the scroll position on every frame.

The container must be the **sole scroll owner** — nothing else (CSS, Bootstrap, the browser) should also be controlling scroll on that element or its ancestors.

---

## Quick Start (Copy-Paste Template)

### 1. Import

```tsx
import { useSmoothScroll } from './Hooks/useSmoothScroll';
```

### 2. Hook Setup

```tsx
const { ref: smoothScrollRef, scrollToTop, Scrollbar } = useSmoothScroll({
  friction: 0.92,
  sensitivity: 1.2,
  disabled: false, // set to true to revert to native scroll
});
```

### 3. JSX Structure

```tsx
{/* Outer wrapper — becomes the positioning context */}
<div className="my-scroll-wrapper">

  {/* Scroll container — the hook controls this */}
  <div ref={smoothScrollRef} className="my-scroll-container">
    {/* Your content here */}
  </div>

  {/* Custom scrollbar — must be a sibling, not inside the scroll container */}
  <Scrollbar />
</div>
```

### 4. Required CSS

```css
/* Wrapper: positioning context with defined size */
.my-scroll-wrapper {
  position: relative;
  overflow: hidden;
  /* MUST have a defined height — one of these: */
  height: 400px;           /* fixed */
  /* OR */ flex: 1 1 auto; min-height: 0;  /* flex child */
  /* OR */ height: 100vh;  /* viewport */
}

/* Scroll container: fills wrapper via absolute positioning */
.my-scroll-container {
  position: absolute;
  inset: 0;
  overflow-y: scroll;
  padding: 20px 24px;      /* padding goes HERE, not on wrapper */
}

/* Hide native scrollbar */
.my-scroll-container::-webkit-scrollbar { display: none; }
.my-scroll-container { scrollbar-width: none; }

/* Custom scrollbar styles */
.ss-scrollbar-track {
  position: absolute;
  top: 8px;
  right: 3px;
  bottom: 8px;
  width: 6px;
  border-radius: 3px;
  z-index: 10;
  opacity: 0;
  transition: opacity 0.3s ease, width 0.2s ease;
  cursor: pointer;
}
.ss-scrollbar-track--visible { opacity: 1; }
.ss-scrollbar-track:hover {
  width: 8px;
  background: rgba(71, 148, 231, 0.06);
}
.ss-scrollbar-thumb {
  width: 100%;
  border-radius: 3px;
  background: rgba(71, 148, 231, 0.25);
  transition: background 0.2s ease;
  cursor: grab;
  min-height: 36px;
}
.ss-scrollbar-thumb:hover { background: rgba(71, 148, 231, 0.45); }
.ss-scrollbar-thumb:active {
  background: rgba(71, 148, 231, 0.6);
  cursor: grabbing;
}
```

---

## Integration Patterns

### Pattern A: Inside a Modal (e.g. Bootstrap Modal)

**Remove `scrollable` from the Modal** — the hook owns scrolling.

```tsx
<Modal show={show} onHide={onHide} size="xl" centered
  dialogClassName="my-modal-dialog"
  contentClassName="my-modal-content"
  // ← NO scrollable prop
>
  <Modal.Header>...</Modal.Header>
  <Modal.Body className="my-modal-body">
    <div ref={smoothScrollRef} className="my-scroll-container">
      {/* content */}
    </div>
    <Scrollbar />
  </Modal.Body>
  <Modal.Footer>...</Modal.Footer>
</Modal>
```

**Modal CSS — the flex layout is critical:**

```css
.my-modal-content {
  display: flex;
  flex-direction: column;
  height: 90vh;             /* MUST be height, NOT max-height */
}

.my-modal-body {
  flex: 1 1 auto;
  min-height: 0;            /* allows flex child to shrink */
  overflow: hidden;          /* body does NOT scroll */
  position: relative;        /* positioning context for absolute child */
  padding: 0;               /* padding moves to scroll container */
}
```

!!! tip "Why `height` not `max-height`"
    The scroll container uses `position: absolute; inset: 0` which resolves against the parent's actual rendered box. `max-height` only sets an upper bound — without content in-flow (absolute child is out of flow), the body collapses. `height` gives a definite size for flex to distribute.

### Pattern B: Full Page

```tsx
<div className="page-layout">
  <header className="page-header">...</header>
  <main className="page-main">
    <div ref={smoothScrollRef} className="page-scroll-container">
      {/* content */}
    </div>
    <Scrollbar />
  </main>
  <footer className="page-footer">...</footer>
</div>
```

```css
.page-layout {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.page-header, .page-footer { flex-shrink: 0; }
.page-main {
  flex: 1 1 auto;
  min-height: 0;
  position: relative;
  overflow: hidden;
}
.page-scroll-container {
  position: absolute;
  inset: 0;
  overflow-y: scroll;
  padding: 20px;
}
.page-scroll-container::-webkit-scrollbar { display: none; }
.page-scroll-container { scrollbar-width: none; }
```

### Pattern C: Sidebar or Panel

```tsx
<aside className="sidebar">
  <div ref={smoothScrollRef} className="sidebar-scroll">
    {/* nav items */}
  </div>
  <Scrollbar />
</aside>
```

```css
.sidebar {
  width: 280px;
  height: 100%;
  position: relative;
  overflow: hidden;
}
.sidebar-scroll {
  position: absolute;
  inset: 0;
  overflow-y: scroll;
  padding: 16px;
}
.sidebar-scroll::-webkit-scrollbar { display: none; }
.sidebar-scroll { scrollbar-width: none; }
```

---

## Hook Options Reference

| Option            | Default | Description                                                        |
|-------------------|---------|--------------------------------------------------------------------|
| `friction`        | `0.92`  | 0–1. Higher = more glide. 0.96 = ice, 0.85 = heavy, 0.92 = silk    |
| `sensitivity`     | `1.2`   | Multiplier on wheel delta. Higher = faster scroll per tick         |
| `maxVelocity`     | `120`   | Cap in px/frame so it can't fly off                                |
| `touchMultiplier` | `1.5`   | Touch drag sensitivity                                             |
| `lerp`            | `0.12`  | Position interpolation factor. Lower = smoother but laggier        |
| `disabled`        | `false` | Set `true` to fully disable (native scroll, no listeners)          |

### Presets

```tsx
// Silk (recommended default)
{ friction: 0.92, sensitivity: 1.2 }

// Ice (long coast, elegant for reading)
{ friction: 0.96, sensitivity: 1.0 }

// Heavy (controlled, snappy)
{ friction: 0.85, sensitivity: 1.5 }
```

---

## API

```tsx
const { ref, scrollTo, scrollToTop, Scrollbar } = useSmoothScroll(options);
```

| Return        | Type                                          | Usage                                              |
|---------------|-----------------------------------------------|-----------------------------------------------------|
| `ref`         | `MutableRefObject<HTMLDivElement>`            | Attach to scroll container div                      |
| `scrollTo`    | `(y: number, immediate?: boolean) => void`    | Scroll to position. `immediate` skips animation     |
| `scrollToTop` | `(immediate?: boolean) => void`               | Shorthand for `scrollTo(0)`                         |
| `Scrollbar`   | `FC<{ className?: string }>`                  | Custom scrollbar component. Render as sibling       |

---

## CSS Rules — Do's and Don'ts

### The wrapper (parent of scroll container)

| DO ✅ | DON'T ❌ |
|--------|----------|
| `position: relative` | `overflow: auto` or `overflow-y: auto` |
| `overflow: hidden` | `scroll-behavior: smooth` |
| Defined height (`height`, `flex: 1 + min-height: 0`) | `max-height` alone (body collapses) |
| `padding: 0` (move padding to scroll container) | `-webkit-overflow-scrolling: touch` |

### The scroll container (where the ref goes)

| DO ✅ | DON'T ❌ |
|--------|----------|
| `position: absolute; inset: 0` | `height: 100%` (doesn't resolve in flex) |
| `overflow-y: scroll` | `overflow: hidden` (hook sets this itself) |
| `padding` for content spacing | `scroll-behavior: smooth` (fights the hook) |
| Hide native scrollbar (see CSS above) | `overscroll-behavior: contain` on the wrapper |

### Anywhere in the ancestor chain

| DO ✅ | DON'T ❌ |
|--------|----------|
| Keep ancestor overflow normal | Any ancestor with `overflow-y: auto/scroll` that wraps the scroll container — creates competing scroll contexts |
| | `will-change: scroll-position` on the container (forces giant compositing layer) |
| | `backdrop-filter: blur()` on the modal shell (GPU re-composite every frame) |
| | `scroll-behavior: smooth` on `html` or `body` if the scroll container is on the page |

---

## Common Pitfalls & Fixes

!!! bug "I can't scroll at all"
    **Cause:** `height: 100%` on the scroll container doesn't resolve because the parent's height comes from flex, not an explicit `height`.
    **Fix:** Use `position: absolute; inset: 0` on the scroll container instead.

!!! bug "The whole page/modal scrolls, header and footer move"
    **Cause:** Bootstrap's `scrollable` prop or `overflow-y: auto` on Modal.Body creates a competing native scroll container.
    **Fix:** Remove `scrollable` from Modal. Set `overflow: hidden` on the body/wrapper. Only the inner scroll container should have `overflow-y: scroll`.

!!! bug "Friction/sensitivity changes have no effect"
    **Cause:** `friction` was set below 0.5 (kills momentum instantly) or another scroll container is handling the events before the hook's container.
    **Fix:** Friction should be 0.85–0.96 for noticeable effect. Check no ancestor is also scrollable.

!!! bug "The modal body collapses to tiny height"
    **Cause:** `max-height: 90vh` instead of `height: 90vh` on the modal content. The absolute scroll container is out of flow so nothing pushes the body open.
    **Fix:** Use `height: 90vh` on the modal content element.

!!! bug "Scrollbar doesn't appear"
    **Cause:** `<Scrollbar />` is inside the scroll container instead of beside it, or the wrapper lacks `position: relative`.
    **Fix:** `<Scrollbar />` must be a **sibling** of the scroll container div, and their shared parent needs `position: relative`.

!!! bug "Content is clipped or has double padding"
    **Cause:** Padding on both the wrapper and scroll container.
    **Fix:** Move all padding to the scroll container. Wrapper gets `padding: 0`.

!!! bug "Dropdowns/popovers get cut off"
    **Cause:** `overflow: hidden` on the wrapper clips absolutely-positioned children.
    **Fix:** Use portals (e.g. React Bootstrap's `container` prop) for dropdowns that need to escape the scroll container, or set appropriate `z-index`.

!!! bug "Scroll position resets on re-render"
    **Cause:** The hook syncs refs on mount. If the container unmounts/remounts (e.g. conditional rendering), scroll position is lost.
    **Fix:** Use `disabled` prop instead of unmounting. If you must unmount, save `scrollTo` position in a ref and restore with `scrollTo(saved, true)` on remount.

---

## Disabling for Accessibility

Always respect `prefers-reduced-motion`:

```tsx
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

const { ref, Scrollbar } = useSmoothScroll({
  friction: 0.92,
  sensitivity: 1.2,
  disabled: prefersReduced, // falls back to native scroll
});
```

When `disabled` is `true`, the hook removes all listeners and restores native scrolling. The `<Scrollbar />` component still works — it reads `scrollTop` regardless of what's driving it.

---

## Checklist for New Integration

- [ ] Import `useSmoothScroll` and call it with options
- [ ] Wrapper has `position: relative`, `overflow: hidden`, and a **defined height**
- [ ] Scroll container uses `position: absolute; inset: 0`
- [ ] Scroll container has `overflow-y: scroll` and padding
- [ ] Native scrollbar is hidden via CSS
- [ ] `<Scrollbar />` is rendered as a **sibling** of the scroll container
- [ ] No `scrollable` on Bootstrap Modal
- [ ] No `scroll-behavior: smooth` anywhere in the chain
- [ ] No `overflow-y: auto/scroll` on wrapper or ancestors
- [ ] No `will-change: scroll-position` on the container
- [ ] No `-webkit-overflow-scrolling: touch` (hook manages momentum)
- [ ] `disabled` prop used rather than conditional rendering to toggle
- [ ] Padding lives on the scroll container, not the wrapper
- [ ] Modal content uses `height: Xvh`, not `max-height`