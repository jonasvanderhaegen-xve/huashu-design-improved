# Interactive States: 8-State Design System

> Essential for app prototypes and interactive demos. A UI element that only has a default state is unfinished. All interactive elements need the full state vocabulary — this is the difference between a static mockup and a believable prototype.

**When to load:** building app prototypes, interactive forms, dashboards with clickable elements, any demo where the user interacts with UI controls.

---

## The 8 States

Every interactive element (button, input, link, card, checkbox, toggle) should be designed across all applicable states.

| State | When it appears | Visual signal |
|---|---|---|
| **Default** | Element is available, not interacted with | Base styling |
| **Hover** | Pointer is over the element | Subtle background shift, cursor change |
| **Focus** | Element is keyboard-selected (Tab) | Visible focus ring |
| **Active** | Element is being pressed (mousedown / touch) | Pressed-in appearance, scale-down |
| **Disabled** | Element is unavailable | Reduced opacity, no interaction cursor |
| **Loading** | Async action triggered, awaiting result | Spinner or skeleton |
| **Error** | Invalid input or action failed | Red indicator, error message |
| **Success** | Action completed | Confirmation color/icon, brief and auto-dismisses |

---

## CSS Patterns

### Hover

```css
.button:hover:not(:disabled) {
  background: color-mix(in oklch, var(--bg) 85%, black);
  transition: background 150ms ease;
}
```

Never add hover styles without `:not(:disabled)` — disabled elements should never show hover feedback.

### Focus (keyboard-accessible ring)

```css
/* Never remove the focus ring — only enhance it */
.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* Remove default ring for mouse users only */
.button:focus:not(:focus-visible) {
  outline: none;
}
```

`:focus-visible` fires only for keyboard navigation, not mouse clicks. This keeps the ring for keyboard users who need it while hiding it for mouse users who don't expect it. **Never use `:focus { outline: none }` without `:focus-visible` replacement.**

### Active (pressed)

```css
.button:active:not(:disabled) {
  transform: scale(0.97);
  transition: transform 80ms ease;
}
```

The active transition should be faster than hover (80ms vs 150ms) — it needs to feel instantaneous.

### Disabled

```css
.button:disabled,
.button[aria-disabled="true"] {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}
```

Use `opacity: 0.4` rather than a custom disabled color — it automatically works against any background. Add `pointer-events: none` to prevent tooltip triggers on disabled elements.

### Loading (inline)

```css
.button--loading {
  position: relative;
  color: transparent; /* hide label while loading */
  pointer-events: none;
}

.button--loading::after {
  content: '';
  position: absolute;
  inset: 0;
  margin: auto;
  width: 16px; height: 16px;
  border: 2px solid currentColor;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 600ms linear infinite;
}
```

Keep the button the same size during loading — don't collapse it to a spinner only. The layout shouldn't shift.

### Error (form input)

```css
.input--error {
  border-color: var(--color-error, #e53e3e);
  box-shadow: 0 0 0 3px oklch(from var(--color-error) l c h / 0.15);
}

.input-error-message {
  color: var(--color-error);
  font-size: 0.875rem;
  margin-top: 4px;
}
```

Error state on the field itself + inline message below the field. Never use only a toast — the user needs to know which field caused the error.

### Success (transient)

```css
.button--success {
  background: var(--color-success, #38a169);
  transition: background 200ms ease;
}
```

Auto-dismiss success state after 2–3 seconds by reverting to default. Don't leave elements permanently in "success" — it stops meaning anything.

---

## Semantic Token Naming

Name tokens by role, not by value. This makes state management coherent as the design evolves.

| ❌ Value-based | ✅ Role-based |
|---|---|
| `--color-red-600` | `--color-error` |
| `--color-green-500` | `--color-success` |
| `--color-yellow-400` | `--color-warning` |
| `--color-blue-500` | `--color-primary` |
| `--spacing-8` | `--space-xs` |
| `--spacing-16` | `--space-sm` |
| `--spacing-32` | `--space-md` |

**Why:** when you change the error color from red to orange, you update `--color-error` in one place. With `--color-red-600`, you chase every usage.

Semantic names for interactive states:

```css
:root {
  /* semantic surface tokens */
  --surface-default: #ffffff;
  --surface-raised: #f8f8f8;
  --surface-hover: #f0f0f0;
  --surface-pressed: #e8e8e8;
  --surface-disabled: #f5f5f5;

  /* semantic state tokens */
  --color-primary: oklch(0.55 0.18 250);
  --color-error: oklch(0.50 0.22 25);
  --color-warning: oklch(0.70 0.18 75);
  --color-success: oklch(0.55 0.16 145);
  --color-info: oklch(0.55 0.14 230);
}
```

---

## Form Validation Timing

| Trigger | When to use |
|---|---|
| **On blur** (field loses focus) | Primary rule — validate after the user finishes, not while typing |
| **On submit** | Catch any untouched fields at submission time |
| **On change** | Only for confirming a valid pattern in real-time (e.g. password strength meter — not an error, just feedback) |

**Never validate on every keystroke for errors.** Showing "invalid email" while the user is still typing "@gmail.co" is aggressive and wrong. Validate on blur.

```javascript
input.addEventListener('blur', () => validateField(input));
form.addEventListener('submit', (e) => {
  if (!validateAll()) e.preventDefault();
});
```

---

## State Coverage Checklist (for demos)

Before delivering an interactive prototype, verify each interactive element has:

- [ ] Default state visible
- [ ] Hover: subtle visual change
- [ ] Focus: visible ring (test with Tab key)
- [ ] Active: pressed feedback (test with mousedown)
- [ ] Disabled: at least one example shown (if relevant to the flow)
- [ ] Loading: shown on async triggers (submit, fetch, upload)
- [ ] Error: shown for at least one form field
- [ ] Success: at least one completion confirmation

A prototype that only shows the happy path in default state is a wireframe, not a prototype.

---

## Prototype Scope vs Production Scope

| Concern | Prototype | Production |
|---|---|---|
| All 8 states on every element | Spot-demonstrate key interactions | Required everywhere |
| Keyboard navigation | Test Tab order on primary flow | Full keyboard-accessible app |
| Screen reader support | Out of scope | Required |
| Touch states on mobile | Tap feedback on primary CTA | Full touch state design |

For huashu demos: demonstrate at least 4 states (default, hover, active, error or loading) on the primary interactive element. The other states can be hinted in code without being fully built out.
