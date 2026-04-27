# Apple Gallery Showcase · Gallery Wall Animation Style

> Inspired by: the Claude Design homepage hero video + Apple product page "work wall" display style
> Original source: huashu-design launch hero v5
> Use cases: **product launch hero animation, skill capability demo, portfolio showcase** — any scenario requiring simultaneous display of multiple high-quality outputs while guiding the audience's attention

---

## When to Use This Style

**Fits:**
- You have 10+ real outputs to display simultaneously on screen (PPT, app, web, infographic)
- The audience is professional (developers, designers, product managers) and sensitive to "craft"
- The intended aesthetic is "restrained, gallery-like, high-end, spacious"
- Focus and overview need to coexist (detail visible without losing the whole)

**Does not fit:**
- Single-product focus (use frontend-design's product hero template)
- Emotional or strongly narrative animation (use timeline narrative template)
- Small screen / portrait (the tilted perspective blurs on small frames)

---

## Core Visual Tokens

```css
:root {
  /* light gallery palette */
  --bg:         #F5F5F7;   /* main canvas background — Apple website gray */
  --bg-warm:    #FAF9F5;   /* warm off-white variant */
  --ink:        #1D1D1F;   /* main text color */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* card 1px border */
  --accent:     #D97757;   /* terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key principles:**
1. **Never use pure black background.** Black backgrounds make work look like film, not like "usable deliverables"
2. **Terracotta orange is the only accent hue** — everything else is grayscale + white
3. **Three-font stack** (serif EN + serif CN + sans + mono) creates a "publication" quality rather than "web product"

---

## Core Layout Patterns

### 1. Floating Cards (the basic unit of this style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* padding acts as a matte */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* primary shadow, soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* second layer near-light, creates floating feel */
  aspect-ratio: 16 / 9;                  /* unified slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* slightly smaller than card radius, visual nesting */
}
```

**Anti-example:** Don't tile flush-to-edge (no padding, no border, no shadow) — that's infographic density, not a gallery.

### 2. 3D Tilted Work Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* deeper perspective, tilt not overdone */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* canvas = 2.25× viewport */
  height: 2520px;                        /* leave room for pan */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* tilt backward */
             rotateY(-10deg)             /* rotate left */
             rotateZ(-2deg);             /* slight tilt, removes excessive rigidity */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spot:**
- rotateX: 10–15deg (any more looks like a cocktail party VIP backdrop)
- rotateY: ±8–12deg (sense of left-right symmetry)
- rotateZ: ±2–3deg ('not machine-placed' — human touch)
- perspective: 2000–2800px (below 2000 produces fisheye, above 3000 approaches orthographic)

### 3. 2×2 Four-Corner Convergence (selection scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides from its corresponding corner (tl/tr/bl/br) toward the center + fade in. The `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Patterns

### Pattern A · Four-Corner Convergence (0.8–1.2s)

4 elements slide in from the four corners of the viewport, simultaneously scaling 0.85→1.0 with ease-out. Good for an opening that "shows choices from multiple directions".

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Pattern B · Selected Card Enlarges + Others Slide Out (0.8s)

The selected card scales 1.0→1.28; unselected cards fade out + blur + drift back toward their corners:

```js
// selected
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// unselected
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key:** Unselected cards need blur — not just fade. Blur simulates depth of field, visually "pushing out" the selected card.

### Pattern C · Ripple Expansion (1.7s)

From center outward, delayed by distance — each card fades in + scales from 1.25x down to 0.94x ("camera pull back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// simultaneously global scale 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Pattern D · Sinusoidal Pan (Continuous Drift)

Combines sine wave + linear drift to avoid the "has a start and end point" loop feeling of a marquee:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // horizontal leftward drift
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // vertical upward drift
const clampedX = Math.max(-900, Math.min(900, panX));   // prevent edge exposure
```

**Parameters:**
- Sine period `0.09–0.15 rad/s` (slow — about 30–50 seconds per oscillation)
- Linear drift `5–8 px/s` (slower than a blink)
- Amplitude `120–220 px` (large enough to feel, small enough not to disorient)

### Pattern E · Focus Overlay (Focus Switch)

**Key design:** The focus overlay is a **flat element** (not tilted), floating above the tilted canvas. The selected slide scales from tile position (approx 400×225) to screen center (960×540), while the background canvas does not change tilt but **darkens to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// background cards dim but remain visible (crucial — don't 100% mask)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Sharpness rules:**
- The focus overlay's `<img>` must `src` to the original image directly — **never reuse compressed gallery thumbnails**
- Preload all original images into a `new Image()[]` array in advance
- The overlay's `width/height` is computed per frame — the browser resamples the original image each frame

---

## Timeline Architecture (Reusable Skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// core easing
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// single render(t) function reads timestamp and writes all elements
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architecture essence:** **All state is derived from timestamp t** — no state machine, no setTimeout. This means:
- Jumping to any moment via `window.__setTime(12.3)` works immediately (convenient for Playwright frame-by-frame screenshots)
- Loop is naturally seamless (t mod DURATION)
- Can freeze any frame during debugging

---

## Craft Details (Easy to Miss, Fatally Important)

### 1. SVG noise texture

A light background's greatest enemy is "being too flat". Overlay an extremely faint fractalNoise layer:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

It looks the same — until you remove it.

### 2. Corner Brand Label

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Shown only during the work wall scene, with fade in/out. Like a museum exhibition label.

### 3. Closing Brand Wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* negative tracking is key — makes type compact into a logotype */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* accent character is actually lighter — creates visual contrast */
}
```

`letter-spacing: -0.045em` is the standard approach for large type on Apple product pages.

---

## Common Failure Patterns

| Symptom | Cause | Fix |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| Tilt looks cheap | Only rotateY, no rotateZ | Add ±2–3deg rotateZ to break rigidity |
| Pan feels "choppy" | Using setTimeout or looping CSS keyframes | Use rAF + sin/cos continuous functions |
| Text unreadable during Focus | Reusing low-resolution gallery tiles | Separate overlay + original image src directly |
| Background too empty | Solid color `#F5F5F7` | Overlay SVG fractalNoise at 0.5 opacity |
| Typeface too "web-generic" | Only Inter | Add Serif (one EN, one CN) + mono — three-stack |

---

## References

- Full implementation sample: local copy at `hero-animation-v5.html` in the 2026-04 huashu-design launch folder
- Original inspiration: claude.ai/design hero video
- Reference aesthetics: Apple product pages, Dribbble shot collection pages

For any animation requirement involving "multiple high-quality outputs to display", copy the skeleton from this file, swap the content + adjust timing.
