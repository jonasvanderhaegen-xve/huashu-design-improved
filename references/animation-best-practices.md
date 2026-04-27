# Animation Best Practices · Motion Design Syntax

> Based on a deep breakdown of three official Anthropic product animations
> (Claude Design / Claude Code Desktop / Claude for Word), distilled into
> "Anthropic-caliber" motion design rules.
>
> Pair with `animation-pitfalls.md` (the bug checklist) — this file is "**do it this way**",
> pitfalls is "**don't do it that way**". They're orthogonal; read both.
>
> **Scope constraint**: this file covers only **motion logic and expressive style** — no specific brand color values.
> Color decisions go through §1.a Core Asset Protocol (extracted from brand spec) or Design Direction Advisor
> (20 philosophies, each with its own palette). This reference discusses **how to move**, not **what color to use**.

---

## §0 · Who You Are · Identity and Taste

> Read this section before any technical rules below. Rules **emerge from identity** —
> not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When you animate, you're not adjusting CSS transitions — you're using digital elements to **simulate a physical world**, so that the audience's subconscious believes "this has weight, inertia, and can overflow."

You don't make PowerPoint animations. You don't make "fade in fade out" animations. Your animations **make people believe the screen is a space they can reach into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not easing curves**
   `linear` is a number; `expoOut` is an object. You treat the pixels on screen as if they deserve to be treated as "things."
   Every easing choice is answering a physics question: "How heavy is this element? What's its friction coefficient?"

2. **Timing allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing rhythm. **An animation with even timing is a tech demo; an animation with rhythm is storytelling.**
   Slowing down at the right moment is more important than using the right easing at the wrong moment.

3. **Giving space to the audience is harder than showing off**
   A 0.5-second pause before a key result is **craft**, not compromise. **Giving the human brain reaction time is the highest skill of an animator.**
   AI defaults to producing animations with no pauses and full information density — that's a beginner move. Your job is restraint.

### §0.3 Taste Standard · What Is Beautiful

Your criteria for "good" versus "great." Each dimension has an **identification method** — when evaluating a candidate animation, use these questions rather than mechanically checking rules.

| Aesthetic dimension | Identification method (audience reaction) |
|---|---|
| **Physical weight** | When the animation ends, elements **land** — they don't just **stop**. The audience subconsciously feels "this has mass" |
| **Audience courtesy** | There's a perceptible pause (≥300ms) before key information arrives — the audience has time to **see it** before continuing |
| **Negative space** | The ending is a hard cut + hold, not a fade to black. The final frame is clear, definitive, decided |
| **Restraint** | Only one moment of "120% precision" in the whole piece; the rest is 80% exactly-right — **showing off everywhere is a cheap signal** |
| **Physical feel** | Curves (not straight lines), irregular timing (not mechanical `setInterval` rhythm), a sense of breathing |
| **Authenticity** | Show the tweaking process, show the bug being fixed — **don't hide the work, don't do "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-Check · The First-Reaction Test

When you finish an animation, **what is the audience's first reaction?** — that's the only metric you're optimizing for.

| Audience reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Competent but unremarkable — you're making PowerPoint |
| "That animation really flows" | good+ | Technically right, but no surprise |
| "This actually looks like it's **floating off the screen**" | great | You've touched physical weight |
| "This doesn't look like AI made it" | great+ | You've reached Anthropic's threshold |
| "I want to **screenshot this** and share it" | great++ | You made something the audience wants to spread |

**The gap between great and good is not technical correctness — it's taste judgment.** Technical correctness + right taste = great. Technical correctness + empty taste = good. Technical errors = not in the door yet.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1–§8 below are the **execution tools** of this identity in specific contexts — not an independent rule list.

- Situation not covered by the rules → go back to §0, judge by **identity**, don't guess
- Rules conflict → go back to §0, use **taste standard** to decide which matters more
- Want to break a rule → first answer: "Which dimension in §0.3 does doing it this way serve?" If you can answer, break it. If not, don't.

Now. Keep reading.

---

## Overview · Animation as Physics: Three Layers

The reason most AI-generated animations feel cheap is that **they behave like "numbers" rather than "objects"**.
Real-world objects have mass, inertia, elasticity, and can overflow. The "premium feel" of the three Anthropic pieces comes from giving digital elements a set of **physical-world motion rules**.

These rules have 3 layers:

1. **Narrative rhythm layer**: Slow-Fast-Boom-Stop timing allocation
2. **Motion curve layer**: Expo Out / Overshoot / Spring — refuse `linear`
3. **Expression language layer**: show process, mouse arcs, Logo morph convergence

---

## 1. Narrative Rhythm · Slow-Fast-Boom-Stop 5-Segment Structure

All three Anthropic pieces follow this structure without exception:

| Segment | Share | Pace | Purpose |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Give humans reaction time, establish authenticity |
| **S2 Generate** | ~15% | Medium | Visual highlight appears |
| **S3 Process** | ~40% | Fast | Show control / density / detail |
| **S4 Boom** | ~20% | Boom | Camera pulls out / 3D pop-out / multi-panel surge |
| **S5 Land** | ~10% | Still | Brand logo + hard cut |

**Timing example** (15-second animation):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Boom 3s · S5 Land 2s

**What not to do:**
- ❌ Even rhythm (same information density every second) — audience fatigue
- ❌ Sustained high density — no peak, no memorable moment
- ❌ Fade-out ending — should **cut hard**

**Self-check**: sketch 5 thumbnails on paper, each representing the climax frame of one segment. If the 5 frames look similar, the rhythm didn't land.

---

## 2. Easing Philosophy · Refuse Linear, Embrace Physics

All animations in the three Anthropic pieces use Bézier curves with a "damped" quality. The default cubic `easeOut` (`1-(1-t)³`) **isn't sharp enough** — too slow to start, too soft to stop.

### Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · fast start, gradual brake (most used — default primary easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · elastic toggle / button pop
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · geometric element settling, natural landing
Easing.spring(t)
```

### Usage Mapping

| Situation | Easing |
|---|---|
| Card rise-in / panel entrance / Terminal fade / focus overlay | **`expoOut`** (primary easing, most frequent) |
| Toggle / button pop / emphasis interaction | `overshoot` |
| Preview geometry settling / physics landing / UI bounce | `spring` |
| Continuous motion (e.g. mouse path interpolation) | `easeInOut` (preserves symmetry) |

### Counter-Intuitive Insight

Most product demo animations are **too fast and too hard**. `linear` makes digital elements feel mechanical; `easeOut` is the baseline; `expoOut` is the technical root of "premium feel" — it gives digital elements a **physical-world sense of weight**.

---

## 3. Motion Language · 8 Shared Principles

### 3.1 Background: No Pure Black or Pure White

None of the three Anthropic pieces uses `#FFFFFF` or `#000000` as the primary background. **Color-tinted neutrals** (warm or cool) have the material feel of "paper / canvas / desktop" — they reduce the machine feel.

**Specific color value decisions** go through §1.a Core Asset Protocol (extracted from brand spec) or Design Direction Advisor (20 philosophies, each with its own background scheme). This reference doesn't prescribe specific values — that's a **brand decision**, not a motion rule.

### 3.2 Easing Is Never Linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process," Not the "Magic Result"

- Claude Design shows tweaking parameters, dragging sliders (not instant-perfect generation)
- Claude Code shows code errors + AI fixing them (not first-attempt success)
- Claude for Word shows the Redline track-changes process (not the finished draft delivered directly)

**Shared subtext**: the product is a **collaborator, pair engineer, senior editor** — not a one-button magician. This precisely addresses the professional user's pain points around controllability and authenticity.

**Anti-AI-slop**: AI defaults to "magic one-click success" animations (generate → perfect result). **Do the opposite** — show process, show tweaking, show bugs being fixed — that's where brand recognition comes from.

### 3.5 Mouse Paths Hand-Crafted (Arc + Perlin Noise)

Real mouse movement isn't a straight line — it's "accelerating start → arc → decelerating correction → click." AI-interpolated straight-line mouse paths **register as uncanny subconsciously**.

```js
// Quadratic Bézier interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → offset midpoint → end (creates the arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Add tiny Perlin Noise (±2px) for "hand tremor"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Convergence"

In all three Anthropic pieces, the logo entrance is **never a simple fade-in** — it **morphs from the previous visual element**.

**Shared pattern**: 1–2 seconds before the end, do a Morph / Rotate / Converge — the whole narrative "collapses" onto the brand mark.

**Low-cost implementation** (no real morph needed):
Have the previous visual element "collapse" to a color block (scale → 0.1, translate toward center);
the color block then "expands" into the wordmark. Use 150ms quick cut + motion blur (`filter: blur(6px)` → `0`).

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scales to 0.1, opacity holds, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo scales from color block center 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-Serif Dual Typography

- **Brand / narration**: serif (conveys "academic / publication / taste")
- **UI / code / data**: sans-serif + monospace

**Using a single typeface throughout is wrong.** Serif gives "taste," sans-serif gives "function."

Specific typeface choices go through brand spec (`brand-spec.md`'s Display / Body / Mono stack) or Design Direction Advisor's 20 philosophies. This reference doesn't prescribe specific fonts — that's a **brand decision**.

### 3.8 Focus Switch = Background Dim + Foreground Sharpen + Flash Guide

Focus switching is **not just** lowering opacity. The full recipe:

```js
// Filter combination for non-focus elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← key: blur is what makes it actually "recede"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus lands, flash highlight at the focus point to guide the eye back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is required**: with only opacity + brightness, out-of-focus elements are still *sharp* — there's no visual sense of "receding to the background." `blur(4–8px)` makes non-focus elements genuinely step back a depth layer.

---

## 4. Specific Motion Techniques (copy-ready code snippets)

### 4.1 FLIP / Shared Element Transition

A button "expanding" into an input field is **not** the button disappearing + a new panel appearing. The core is **the same DOM element** transitioning between two states — not two elements cross-fading.

```jsx
// With Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ After click, same layoutId
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

Native implementation reference: https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expand (width → height)

A panel expanding is **not** stretching width and height simultaneously. Instead:
- First 40% of time: only stretch width (height stays small)
- Last 60% of time: width holds, height fills in

This simulates the physical world's "spread out first, then fill" feeling.

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

Table rows, card grids, list items entering — **each element delays 30ms**, with `translateY` returning from 10px to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-linear Breathing · 0.5s Hover Before Key Results

Machines execute fast and continuously, but **holding 0.5 seconds before a key result appears** gives the human brain reaction time.

```jsx
// Typical scenario: AI finishes generating → hold 0.5s → result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves; let the audience stare at the loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Anti-example**: jumping immediately to the result after AI generation — the audience has no reaction time, information is lost.

### 4.5 Chunk Reveal · Simulating Token Streaming

AI-generated text **should not use `setInterval` to pop out one character at a time** (like old movie subtitles). Use **chunk reveal** — 2–5 characters appear at once, at irregular intervals, simulating real token streaming.

```js
// Reveal by chunk, not by character
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split on words + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40–120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

3 of Disney's 12 principles. Anthropic uses them explicitly:

- **Anticipation**: a small opposing movement before the action starts (button slightly shrinks before popping out)
- **Action**: the main motion itself
- **Follow-through**: residual movement after the action ends (card bounces slightly after landing)

```js
// Full three-phase card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);       // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // main action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);        // bounce settle
// Final scale = apply in sequence or as product
```

**Anti-example**: animations with only Action and no Anticipation + Follow-through look like "PowerPoint animations."

### 4.7 3D Perspective + translateZ Layering

For that "tilted 3D + floating card" quality, add perspective to the container and different `translateZ` values to individual elements:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* eye level slightly overhead */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio:**
- Greater than 10° → elements look too distorted, like they're "falling over"
- Less than 5° → looks like "skew" rather than "perspective"
- The asymmetric 8° × -4° ratio simulates a natural angle of "camera looking down from the upper left of a desk"

### 4.8 Diagonal Pan · Moving X and Y Simultaneously

Camera movement is not pure vertical or pure horizontal — it **moves X and Y simultaneously** to simulate diagonal travel:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y have different frequencies (0.22 vs 0.35) — prevents the Lissajous pattern from becoming regular and mechanical.

---

## 5. Scene Recipes (Three Narrative Templates)

The three reference videos correspond to three product personalities. **Choose the one that best fits your product** — don't mix and match.

### Recipe A · Apple Keynote Dramatic (Claude Design style)

**Best for**: major version launches, hero animations, visual impact priority
**Rhythm**: Slow-Fast-Boom-Stop strong arc
**Easing**: full `expoOut` + occasional `overshoot`
**SFX density**: high (~0.4/s); SFX pitch tuned to BGM scale
**BGM**: IDM / minimal tech electronic — calm + precise
**Convergence**: camera snaps out → drop → Logo morph → ethereal single note → hard cut

### Recipe B · Single-Take Tool-Style (Claude Code style)

**Best for**: developer tools, productivity apps, flow-state scenarios
**Rhythm**: sustained stable flow — no obvious peak
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (BGM alone drives edit rhythm)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85–90 BPM
**Core technique**: key UI actions land on BGM kick/snare transients — "**musical beat = interaction sound**"

### Recipe C · Enterprise Productivity Narrative (Claude for Word style)

**Best for**: enterprise software, documents / spreadsheets / calendar, professionalism priority
**Rhythm**: multi-scene hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panels)
**SFX density**: medium (~0.3/s); UI click-sounds primary
**BGM**: Jazzy Instrumental, minor key, BPM 90–95
**Key highlight**: one scene must be the "whole-piece moment" — 3D pop-out / element floating off the plane

---

## 6. Anti-Patterns · This Is AI Slop

| Anti-pattern | Why it's wrong | Correct approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is `linear`'s cousin; all elements move at the same speed | Use `expoOut` + per-element stagger |
| All entrances are `opacity 0→1` | No sense of directional movement | Pair with `translateY 10→0` + Anticipation |
| Logo fades in | No narrative convergence | Morph / Converge / collapse-and-expand |
| Mouse moves in a straight line | Subconscious machine feel | Bézier arc + Perlin Noise |
| Typing via `setInterval` one character at a time | Looks like old movie subtitles | Chunk Reveal, random intervals |
| No pause before key results | Audience has no reaction time | 0.5s hold before the result |
| Focus switch only changes opacity | Out-of-focus elements stay sharp | opacity + brightness + **blur** |
| Pure black or pure white background | Cyberpunk feel / reflective fatigue | Color-tinted neutral (follow brand spec) |
| All animations at the same speed | No rhythm | Slow-Fast-Boom-Stop |
| Fade-out ending | No decisiveness | Hard cut (hold on the final frame) |

---

## 7. Self-Check Checklist (60 seconds before delivery)

- [ ] Narrative structure is Slow-Fast-Boom-Stop — not even rhythm?
- [ ] Default easing is `expoOut` — not `easeOut` or `linear`?
- [ ] Toggle / button pop uses `overshoot`?
- [ ] Card / list entrances have 30ms stagger?
- [ ] 0.5s hold before key results?
- [ ] Typing uses Chunk Reveal — not `setInterval` one-character?
- [ ] Focus switch includes blur (not just opacity)?
- [ ] Logo converges via morph — not a fade-in?
- [ ] Background is not pure black / pure white (color-tinted)?
- [ ] Typography has serif + sans-serif hierarchy?
- [ ] Ending is a hard cut — not a fade-out?
- [ ] (If mouse is present) Mouse path is an arc — not a straight line?
- [ ] SFX density matches the product personality (see Recipe A/B/C)?
- [ ] BGM and SFX have 6–8dB loudness difference? (see `audio-design-rules.md`)

---

## 8. Relationship to Other References

| Reference | Role | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical bug avoidance (16 rules) | "**Don't do this**" · the inverse of this file |
| `animations.md` | Stage/Sprite engine usage | The basics of **how to write** animations |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **adding audio** to animations |
| `sfx-library.md` | 37-SFX catalog | Sound effect **asset library** |
| `apple-gallery-showcase.md` | Apple gallery display style | Deep dive into one specific motion style |
| **This file** | Positive motion design syntax | "**Do it this way**" |

**Call order:**
1. Start with SKILL.md workflow Step 3's four positioning questions (determine narrative role and visual temperature)
2. Once direction is set, read this file to determine **motion language** (Recipe A/B/C)
3. When writing code, reference `animations.md` and `animation-pitfalls.md`
4. When exporting video, follow `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source Material for This File

- Anthropic official animation breakdown: `BEST-PRACTICES.md` in the project's reference-animations directory
- Anthropic audio breakdown: `AUDIO-BEST-PRACTICES.md` in the same directory
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strict filtering**: no specific brand color values, typeface names, or product names are stored in this reference.
  Color/font decisions go through §1.a Core Asset Protocol or the 20 design philosophies.
