# Gallery Ripple + Multi-Focus · Scene Choreography Philosophy

> A **reusable visual choreography structure** distilled from the huashu-design hero animation v9 (25 seconds, 8 scenes).
> Not an animation production pipeline — but **when this choreography is 'the right call'**.
> Reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-Line Lead

> **When you have 20+ visually cohesive assets and the scene needs to "express scale and depth", reach for Gallery Ripple + Multi-Focus instead of stacking layout elements.**

Generic SaaS feature animations, product launches, skill promotions, portfolio showcases — as long as the asset count is sufficient and the style is consistent, this structure almost always delivers.

---

## What This Technique Actually Communicates

Not "showing off assets" — it tells a narrative through **two rhythmic shifts:**

**Beat 1 · Ripple expansion (~1.5s):** 48 cards radiate outward from the center. The audience is stunned by sheer quantity — "oh, there's this much output from this thing".

**Beat 2 · Multi-Focus (~8s, 4 iterations):** The camera slow-pans while 4 times dimming + desaturating the background and enlarging a single card to screen center. The audience shifts from "impact of quantity" to "gaze at quality" — each iteration a steady 1.7s.

**Core narrative structure: Scale (Ripple) → Gaze (Focus × 4) → Fade (Walloff).** These three beats combine to express "Breadth × Depth" — not just that it produces a lot, but that each one is worth pausing to look at.

Counter-examples for comparison:

| Approach | Audience perception |
|------|---------|
| 48 cards in static layout (no Ripple) | Pretty but no narrative — feels like a grid screenshot |
| One card at a time, fast cuts (no Gallery context) | Feels like a slideshow — loses the sense of scale |
| Ripple without Focus | Stunning but nothing sticks in memory |
| **Ripple + Focus × 4 (this recipe)** | **Overwhelmed by quantity, then gazing at quality, then serene fade — complete emotional arc** |

---

## Prerequisites (All Must Be Met)

This choreography is **not universal** — all 4 conditions below are required:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 makes the Ripple look "sparse" — every slot needs to be moving to create density. v9 used 48 slots × 32 images (looped to fill).

2. **Assets are visually consistent in style**
   All 16:9 slide previews / all app screenshots / all cover designs — aspect ratio, palette, and layout should feel like "a system". Mixing styles makes the Gallery look like a clipboard.

3. **Individual assets remain legible when enlarged**
   Focus enlarges a card to 960px wide. If the source image blurs or carries sparse information when enlarged, the Focus beat fails. Reverse test: can you pick 4 cards from 48 as "most representative"? If you can't, asset quality is uneven.

4. **The scene is landscape or square — not portrait**
   The Gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs horizontal extent — portrait makes the tilt look narrow and awkward.

**Fallback paths when conditions aren't met:**

| Missing condition | Fall back to |
|-------|-----------|
| Assets < 20 | Switch to "3–5 side-by-side static display + individual focus" |
| Inconsistent style | Switch to "cover + 3 chapter hero images" keynote style |
| Sparse information | Switch to "data-driven dashboard" or "key quote + large type" |
| Portrait scene | Switch to "vertical scroll + sticky cards" |

---

## Technical Recipe (v9 Real-World Parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, large overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key:** The canvas is 2.25× larger than the viewport — this gives the pan a "peeking into a larger world" quality.

### Ripple Expansion (Distance-Delay Algorithm)

```js
// each card's entrance time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // offset from center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Key parameters:**
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Maximum delay 0.8s (center exits first, corners last)
- Each card entrance duration 0.7s
- Easing: `expoOut` (burst feeling, not smooth)

**Simultaneous:** canvas scale from 1.25 → 0.94 (zoom out to reveal) — creates a synchronized pull-back feeling as cards appear.

### Multi-Focus (4-Beat Rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm pattern:** Each focus 1.7s, 0.6s breathing gap between. Total 8s (11.0–19.6s).

**Internal structure of each focus:**
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background change (this is the key):**

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not just opacity — simultaneously desaturate + darken.** This makes the foreground overlay's color "pop out" rather than merely "appear brighter".

**Focus overlay size animation:**
- 400×225 on entrance → 960×540 at hold state
- 3 layers of shadow + 3px accent color outline ring — creates a "framed" feeling

### Pan (Continuity Prevents Stagnation)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift, dual-layer motion — not a pure loop; position is different every moment
- X/Y frequencies differ (0.12 vs 0.09) to prevent the audience from perceiving a "regular loop"
- Clamped to ±900/500px to prevent drifting off-screen

**Why not pure linear pan:** With linear pan, the audience can "predict" where the next second will be. Sine + drift makes every second new; combined with the 3D tilt, it produces a slight "sea-legs feeling" (the good kind) that holds attention.

---

## 5 Reusable Patterns (Distilled from v6→v9 Iterations)

### 1. **expoOut as primary easing, not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (burst then rapid convergence).

**Why:** expoOut reaches 90% within the first 30%, which feels more like physical damping — matching the intuition of "a heavy object landing". Especially good for:
- Card entrance (sense of weight)
- Ripple expansion (shockwave)
- Brand float (settling feeling)

**When to still use cubicOut:** focus out ramp, symmetric micro-animations.

### 2. **Paper-texture base color + terracotta orange accent (Anthropic lineage)**

```css
--bg: #F7F4EE;        /* warm paper */
--ink: #1D1D1F;       /* near black */
--accent: #D97757;    /* terracotta orange */
--hairline: #E4DED2;  /* warm hairline */
```

**Why:** The warm base color retains a "breathing quality" after GIF compression, unlike pure white which feels "screen-like". Terracotta orange as the single accent threads through the terminal prompt, selected dir-card, cursor, brand hyphen, and focus ring — all visual anchor points are tied together by this one color.

**v5 lesson:** Adding a noise overlay to simulate "paper texture" destroyed GIF frame compression (every frame differed). v6 switched to "base color + warm shadow only" — paper feel retained at 90%, GIF file size reduced by 60%.

### 3. **Two-tier shadow to simulate depth — no real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

The `sin(i × 1.7) + cos(i × 0.73)` deterministic algorithm assigns each card to near/mid/far shadow tier — **visually creates a "3D stacking" feel, but transform never changes per frame, GPU cost 0**.

**The cost of real 3D:** Each card with its own `translateZ` forces the GPU to compute 48 transforms + shadow blur every frame. v4 tried it — even 25fps Playwright recording struggled. v6's two-tier shadow is <5% perceptually different, but ~10× cheaper.

### 4. **Weight variation (font-variation-settings) is more cinematic than size variation**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

Brand wordmark morphs Thin → Bold over 0.9s, combined with a subtle letter-spacing shift (-0.045 → -0.048em).

**Why it beats scale animation:**
- Scale animation is so familiar the audience's expectations are fixed
- Weight change is an "internal fullness feeling" — like a balloon being inflated, not like something being pushed toward you
- Variable fonts became widespread only after 2020 — audiences subconsciously feel "modern"

**Limitation:** Must use a variable font (Inter / Roboto Flex / Recursive, etc.). Static fonts can only fake it — switching between fixed weights produces jumps.

### 5. **Low-intensity persistent corner branding**

During the Gallery phase, a small `HUASHU · DESIGN` label sits in the top-left corner — 16% opacity, 12px, wide tracking.

**Why include it:**
- After the Ripple burst, audiences easily "lose focus" and forget what they're watching — the corner label provides an anchor
- More refined than a full-screen large logo — brand practitioners know a signature doesn't need to shout
- Leaves an attribution signal when the GIF is screenshotted and shared

**Rule:** Appears only in the middle section (when the frame is busy); hidden at the opening (don't cover the terminal) and the ending (brand reveal is the protagonist).

---

## Counter-examples: When Not to Use This Choreography

**❌ Product demo (showing features):** Gallery makes every card flash by — the audience won't remember any specific feature. Switch to "single-screen focus + tooltip annotation".

**❌ Data-driven content:** The audience needs to read numbers — Gallery's fast pace doesn't allow it. Switch to "data chart + item-by-item reveal".

**❌ Narrative storytelling:** Gallery is a "parallel" structure; stories need "cause and effect". Switch to keynote chapter transitions.

**❌ Only 3–5 assets:** Ripple density is too low — looks like a "patch". Switch to "static layout + individual highlights".

**❌ Portrait (9:16):** 3D tilt needs horizontal extent — portrait makes the tilt feel "crooked" rather than "expansive".

---

## How to Tell If Your Task Suits This Choreography

Three-step quick check:

**Step 1 · Asset count:** Count how many visually cohesive assets you have. < 15 → stop; 15–25 → stretch it; 25+ → go ahead.

**Step 2 · Consistency test:** Place 4 random assets side by side. Do they look like "a system"? If not → unify the style first, or change the approach.

**Step 3 · Narrative fit:** Are you expressing "Breadth × Depth" (quantity × quality)? Or "flow", "features", "story"? If not the former, don't force it.

All three yes — fork the v6 HTML, swap the `SLIDE_FILES` array and timeline and you're done. Change the palette via `--bg / --accent / --ink` to re-skin without changing the structure.

---

## Related References

- Full technical workflow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio setup (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Apple gallery-style lateral reference: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration): `www.huasheng.ai/huashu-design-hero/index.html`
