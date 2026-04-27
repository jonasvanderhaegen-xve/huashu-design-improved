# Cinematic Patterns · Best Practices for Workflow Demos

> 5 key patterns for upgrading from "PPT animations" to "keynote-grade cinematic".
> Distilled from the two cinematic demos (Nuwa workflow + Darwin workflow) in the 2026-04 "Chatting about Skills" deck — tested and reproducible.

---

## 0 · What Problem This Document Solves

When you need to create a "demo animation of a workflow" (typical scenarios: skill workflows, product onboarding, API call flows, agent task execution), there are two common approaches:

| Paradigm | What it looks like | Consequence |
|---|---|---|
| **PPT animation** (bad) | step 1 fade in → step 2 fade in → step 3 fade in, 4 boxes arranged on screen simultaneously | Audience feels "it's just a PPT with fade effects" — no wow moment |
| **Cinematic** (good) | scene-based, focused on one thing at a time, scenes connected by dissolve / focus pull / morph | Audience feels "this is a product keynote clip" — they'll want to screenshot and share it |

The root difference is **not animation technique** — it is **narrative paradigm**. This document explains how to upgrade from the former to the latter.

---

## 1 · Five Core Patterns

### Pattern A · Dashboard + Cinematic Overlay Dual-Layer Structure

**Problem:** A bare cinematic defaults to a black screen + a ▶ button — if the user lands on the slide without clicking, there's nothing to see.

**Solution:**
```
DEFAULT state (always visible): full static workflow dashboard
  └── audience sees at a glance how this skill / workflow runs

▶ triggered (overlay floats up): 22-second cinematic
  └── fades back to DEFAULT automatically on completion

```

**Implementation notes:**
- `.dash` defaults visible; `.cinema` defaults to `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button in the bottom-right (not a large centered overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Run once via `requestAnimationFrame` (not a loop) — `endCinematic()` reverses state on completion

**Anti-pattern:** Default = large centered ▶ overlay covering everything; the page is blank before the user clicks.

---

### Pattern B · Scene-based, NOT Step-based

**Problem:** Breaking the animation into "step 1 appears → step 2 appears → ..." is PPT thinking.

**Solution:** Break into 5 scenes. Each scene is an **independent shot** — full-screen, focused on exactly one thing:

| Scene type | Role | Duration |
|---|---|---|
| 1 · Invoke | User input trigger (terminal typewriter) | 3–4s |
| 2 · Process | Core workflow visualization (distinct visual language) | 5–6s |
| 3 · Result/Insight | Key distilled output (visualized) | 4–5s |
| 4 · Output | Actual deliverable display (file / diff / number) | 3–4s |
| 5 · Hero Reveal | Closing hero moment (large type + value proposition) | 4–5s |

**Total duration ≈ 22 seconds** — this is the tested golden length:
- Under 18 seconds: the PM hasn't settled in before it's over
- Over 25 seconds: they lose patience
- 22 seconds is exactly enough to "hook → unfold → close → leave an impression"

**Implementation notes:**
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` — global timeline
- A single `requestAnimationFrame(render)` runs all scene opacity / transform calculations
- Don't use a setTimeout chain (prone to breaking, hard to debug)
- Easing must use `expoOut` / `easeOut` / cubic-bezier — **linear is forbidden**

---

### Pattern C · Each Demo Must Have Its Own Distinct Visual Language

**Problem:** After finishing the first cinematic, taking a shortcut on the second by reusing the same template (same orbit + pentagon + typewriter + hero large text) and only swapping copy.

**Consequence:** The audience notices the two skills "look identical" — which says "there's no difference between these two skills".

**Solution:** Each workflow has a different core metaphor — the visual language must differ accordingly.

**Comparison case:**

| Dimension | Nuwa (Distiller) | Darwin (Skill Optimizer) |
|---|---|---|
| Core metaphor | Collect → Distill → Write | Loop → Evaluate → Ratchet |
| Visual motion | Float / Radiate / Pentagon | Loop / Ascend / Contrast |
| Scene 2 | 3D Orbit · 8 files floating in a perspective ellipse | Spin Loop · token travels 5 laps around a 6-node ring |
| Scene 3 | Pentagon · 5 tokens radiating from center | v1 vs v5 · side-by-side diff (red version vs gold version) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" serif italic large type | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Test:** Cover the copy and look only at the visuals — can you tell which demo it is? If not, you took a shortcut.

---

### Pattern D · Use Real AI-Generated Assets — Not Emoji or Hand-Drawn SVG

**Problem:** 3D orbit / gallery needs asset fragments floating — emoji (📚🎤) is ugly and brand-less, and hand-drawn SVG book spines never look like real books.

**Solution:** Run `huashu-gpt-image` to generate a 4×2 grid image (8 thematically related objects · white background · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to extract 8 individual transparent PNGs.

**Prompt notes** (detailed prompt patterns in the `huashu-gpt-image` skill):
- IP anchoring ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (easier to cut out; gray background is atmospheric but makes transparent extraction difficult)
- 4×2 not 5×5 (avoids last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern:** Using emoji as icons, or CSS silhouettes instead of product imagery.

---

### Pattern E · BGM + SFX Dual-Track System

**Problem:** Animation without sound makes audiences unconsciously feel "this thing looks like a cheap demo".

**Solution:** Long-play BGM + 11 SFX cues.

**Universal SFX cue recipe** (for workflow demos):

| Time | SFX | Trigger |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter completes, press enter |
| 4.0s | slide-in | Scene 2 elements enter |
| 5–9s × 5 | sparkle | Key process node (each generation / token / data point) |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | Trigger every 2 characters (don't go too dense) |

**Frequency separation:** BGM volume 0.32 (low-frequency bed noise), SFX volume 0.55 (mid-high punch), sparkle 0.7 (needs to be noticeable), logo-reveal 0.85 (strongest hero moment).

**User control:**
- Must have ▶ start overlay (browser autoplay restrictions)
- Small mute button in top-right (user can silence at any time)
- Don't make it auto-play sound on page turn

---

## 2 · Static Dashboard Design Notes

The dashboard is Layer 1 of the dual-layer structure — a PM who never clicks ▶ can still understand the skill.

**Layout:** 3-column grid (or 1 large + 2 small) — each panel answers one question:

| Panel type | What it answers | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is the workflow of this skill?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does the real output data look like?" | Darwin 8-dimension rubric snapshot |
| **Trajectory / Evolution** | "How does it evolve across multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has already been produced?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "Input what → Output what" | Nuwa example strip: `› nuwa distill feynman → feynman.skill (21 min)` |

**Key constraints:**
- Information density must be sufficient (each panel carries distinct information)
- But don't stuff in data slop (every number must mean something)
- Color scheme consistent with cinematic (same palette — switching is seamless)

---

## 3 · Debugging and Dev Tools

Any long animation must come with three dev tools — otherwise debugging becomes a nightmare.

### Tool 1 · `?seek=N` — Freeze to Second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// in render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` — view the frame at second 12 directly, no need to wait for playback.

### Tool 2 · `?autoplay=1` — Skip the ▶ Overlay

Useful for Playwright automated screenshot testing, and for force-starting when embedded in an iframe.

### Tool 3 · Manual REPLAY Button

Small button in the top-right — users and debuggers can replay as many times as needed. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe Embedding Gotchas (when cinematic is embedded in a deck)

### Gotcha 1 · Parent-window click zones intercept buttons inside the iframe

If the deck's index.html has "transparent 22vw click zones on left and right for page-turn", they **overlap the ▶ play button inside the iframe** — a user click on the button gets swallowed as "next page".

**Fix:** Add `top: 12vh; bottom: 25vh` to the click zone — leave the top and bottom 25% unblocked so both the center ▶ and bottom-right ▶ inside the iframe are clickable.

### Gotcha 2 · Keyboard events lost after iframe grabs focus

After the user clicks inside the iframe, focus is inside the iframe — the parent window stops receiving ←/→ keyboard events.

**Fix:**
```js
iframe.addEventListener('load', () => {
  // inject keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // pull focus back to parent after click
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Gotcha 3 · Behavioral differences between file:// and https://

A cinematic that worked on local file:// may break after deployment because:
- `file://` — iframe contentDocument is same-origin
- `https://` — also same-origin (if same host), but audio autoplay restrictions are stricter

**Fix:**
- Before deploying, run `python3 -m http.server` for a local HTTP test
- BGM must wait for a user ▶ click before calling `bgm.play()` — don't autoplay on page load

---

## 5 · Anti-Pattern Quick Reference

| ❌ Anti-pattern | ✅ Correct pattern |
|---|---|
| Default = black-screen ▶ overlay | Default = static dashboard, ▶ is supplementary |
| 4 steps arranged horizontally, all fading in | 5 scenes, full-screen transitions, each focused on one thing |
| Reusing templates with swapped copy for different demos | Each demo has its own visual language (distinguishable without copy) |
| emoji / hand-drawn SVG as assets | gpt-image-2 large grid + extract_grid cutout |
| No BGM, no SFX | BGM + 11 SFX cues, dual-track |
| Scheduling with a setTimeout chain | requestAnimationFrame + global timeline T object |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent click zones | click zone with top/bottom margin to clear buttons |

---

## 6 · Time Budget

Using this pattern set, a full cinematic demo (including dashboard):

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 min (deliberate — this determines visual independence) |
| Dashboard static layout + content | 1 hour |
| Cinematic 5-scene implementation | 1.5 hours |
| Audio cue timing + replay button | 30 min |
| Playwright screenshot validation at 5 key moments | 15 min |
| **Single demo total** | **3–4 hours** |

The second demo reuses the framework but **the visual language must be independent** — approximately 2–3 hours.
