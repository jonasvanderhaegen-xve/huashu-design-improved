# Responsive Design Patterns

> Beyond screen width. Real responsive design accounts for input method, device capabilities, safe areas, and content-driven layout shifts. Load when building device-framed prototypes, mobile-first demos, or any HTML that will be viewed on physical devices.

---

## Input Method Detection (not just screen size)

Screen width tells you nothing about how the user is interacting. A 768px screen might be a tablet with touch, a folded laptop, or a small desktop monitor. Use pointer and hover queries to target the actual input method.

```css
/* Touch-primary devices: no reliable hover, fat-finger targets */
@media (pointer: coarse) {
  --hit-target: 44px;       /* Apple HIG minimum */
  --tap-spacing: 8px;       /* minimum gap between tap targets */
  
  .button { min-height: var(--hit-target); }
  .nav-item { padding: 12px 16px; } /* increase touch area */
  .tooltip { display: none; } /* tooltips don't work on touch */
}

/* Precision pointer (mouse/trackpad): hover is reliable */
@media (pointer: fine) {
  --hit-target: 32px;       /* can afford tighter targets */
  
  .button:hover { /* hover states safe to use */ }
  .tooltip { display: block; } /* tooltips make sense */
}

/* Hybrid device (Surface, iPad with keyboard): both available */
@media (pointer: coarse) and (hover: none) {
  /* pure touch — no hover fallback */
}
```

**Why this matters for prototypes:** iOS app mockups wrapped in `ios_frame.jsx` are viewed on desktop. If the prototype uses hover states as the primary interaction trigger, they won't work on a real device. Pointer queries let you design for the actual delivery target.

---

## Safe Area Handling (iOS notch / Dynamic Island)

Devices with notches, rounded corners, and home indicators clip content that extends into those regions. The CSS `env()` function provides insets.

```css
/* Global safe-area padding — apply to fixed elements and full-bleed containers */
.app-shell {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}

/* Tab bar (pinned to bottom) — must clear home indicator */
.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0; right: 0;
  padding-bottom: max(env(safe-area-inset-bottom), 8px);
  height: calc(49px + env(safe-area-inset-bottom));
}

/* Full-bleed hero that extends behind status bar */
.hero {
  padding-top: calc(env(safe-area-inset-top) + 16px);
}
```

**Viewport meta tag — required:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```
`viewport-fit=cover` allows content to extend under the notch. Without it, `env()` insets are zero and the screen is letterboxed.

**In huashu device frames:** `ios_frame.jsx` handles the outer bezel. For prototypes where the inner HTML simulates a real iOS app, apply safe-area padding to the simulated app shell — not the wrapper.

---

## Viewport Units: dvh vs vh

`100vh` on mobile browsers includes the browser chrome (address bar). Content set to `100vh` gets clipped when the address bar is visible.

```css
/* Old — clips behind browser chrome on mobile */
.hero { height: 100vh; }

/* Modern — dynamic viewport height, accounts for browser chrome */
.hero { height: 100dvh; }

/* Fallback for older browsers */
.hero {
  height: 100vh;            /* fallback */
  height: 100dvh;           /* override where supported */
}

/* Other dynamic units */
.sidebar { height: 100svh; }  /* small viewport (chrome visible) */
.panel   { height: 100lvh; }  /* large viewport (chrome hidden) */
```

---

## Content-Driven Breakpoints

Don't break at arbitrary pixel values (768px, 1024px). Break when your content breaks.

**Process:**
1. Start with mobile layout
2. Expand the viewport slowly
3. When the layout starts to look wrong or awkward — that's your breakpoint
4. Set the breakpoint at that content-specific width, not at a standard value

```css
/* Content-driven — breakpoint is where the two-column layout makes sense */
.feature-grid {
  display: grid;
  grid-template-columns: 1fr;      /* single column on narrow */
}

@media (min-width: 52rem) {        /* ~830px — where the content breathes */
  .feature-grid {
    grid-template-columns: 1fr 1fr;
  }
}
```

**Auto-fit as an alternative to breakpoints:**

```css
/* No breakpoint needed — columns appear automatically when space allows */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
```

`minmax(280px, 1fr)` means: columns are at least 280px, share available space equally. Three columns on wide screens, two on medium, one on mobile — no breakpoints written.

---

## Mobile-First Rhythm

Write CSS from the smallest screen up, not from the largest down. Every element starts with its mobile style; media queries layer on complexity for larger viewports.

```css
/* Mobile-first */
.container {
  padding: 16px;           /* mobile */
  max-width: 100%;
}

@media (min-width: 40rem) {
  .container {
    padding: 24px;         /* tablet */
    max-width: 720px;
    margin: 0 auto;
  }
}

@media (min-width: 64rem) {
  .container {
    padding: 40px 48px;    /* desktop */
    max-width: 1200px;
  }
}
```

**Why mobile-first:**
- Forces you to decide what's actually essential (mobile screen is the constraint)
- `min-width` queries add features; `max-width` queries take them away — adding is easier to reason about
- Mobile is often the primary usage context for prototypes that will be pitched to stakeholders on their phones

---

## Touch Target Checklist

Before delivering any mobile prototype:

- [ ] All tap targets ≥ 44×44px (Apple HIG) / 48×48dp (Material)
- [ ] Minimum 8px gap between adjacent tap targets (prevents mis-taps)
- [ ] No hover-only interactions (use tap/click instead)
- [ ] Tooltips replaced with inline labels or long-press patterns on touch
- [ ] Text inputs: font-size ≥ 16px (prevents iOS auto-zoom on focus)
- [ ] Fixed/sticky elements clear the bottom safe area
- [ ] Scrollable areas have `-webkit-overflow-scrolling: touch` (or `overscroll-behavior: contain` for modern)

---

## Quick Reference: Device Safe Areas

| Device | Top inset | Bottom inset |
|---|---|---|
| iPhone with Dynamic Island | ~59px | ~34px |
| iPhone with notch | ~44px | ~34px |
| iPhone SE (no notch) | ~20px | ~0px |
| iPad (no notch) | ~24px | ~20px |
| Android (varies) | Use `env()` — values differ per manufacturer |

**In practice:** use `env(safe-area-inset-*)` always and let the device report its own values. Don't hardcode inset values.
