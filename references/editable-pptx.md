# Editable PPTX Export: HTML Hard Constraints + Dimension Decisions + Common Errors

This document covers the path of **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element-by-element into real editable PowerPoint text boxes** — the only path `export_deck_pptx.mjs` supports.

> **Core prerequisite**: to use this path, HTML must be written with the 4 constraints below from line 1. **Not written first, then converted** — retroactive remediation triggers 2–3 hours of rework (2026-04-20 stakeholder deck project real incident).
>
> For visual-priority scenarios (animation / web components / CSS gradients / complex SVG) use the PDF path (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`) instead. **Don't expect PPTX export to give you both visual fidelity and editability** — this is a physical constraint of the PPTX file format itself (see "Why the 4 Constraints Are Physical Constraints, Not Bugs" at the end).

---

## Canvas Dimensions: Use 960×540pt (LAYOUT_WIDE)

PPTX units are **inches** (physical dimensions), not px. The principle: the body's `computedStyle` dimensions must **match the presentation layout's inch dimensions** (±0.1", enforced by `html2pptx.js`'s `validateDimensions` check).

### 3 Candidate Dimensions

| HTML body | Physical size | Matching PPT layout | When to use |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommended** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when user specifies "legacy PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard dimension — fonts appear abnormally small when projected |

**Don't think of HTML dimensions as resolution.** PPTX is a vector document; body dimensions determine **physical size**, not clarity. An oversized body (20″×11.25″) doesn't make text crisper — it only makes font pt sizes smaller relative to the canvas, which looks worse when projected or printed.

### Three Equivalent body Declarations

```css
body { width: 960pt;  height: 540pt; }    /* Most explicit — recommended */
body { width: 1280px; height: 720px; }    /* Equivalent, px habit */
body { width: 13.333in; height: 7.5in; }  /* Equivalent, inch intuition */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch — no custom definition needed
```

---

## 4 Hard Constraints (Violations Will Error)

`html2pptx.js` translates HTML DOM elements one-by-one into PowerPoint objects. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: No Bare Text Directly in DIVs — Wrap in `<p>` or `<h1>`–`<h6>`

```html
<!-- ❌ Wrong: text directly in div -->
<div class="title">Q3 Revenue Up 23%</div>

<!-- ✅ Correct: text in <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 Revenue Up 23%</h1></div>
<div class="body"><p>New users are the primary driver</p></div>
```

**Why**: PowerPoint text must exist inside a text frame; text frames correspond to HTML block-level elements (`p`/`h*`/`li`). A bare `<div>` has no corresponding text container in PPTX.

**`<span>` also can't carry primary text** — span is an inline element with no independent alignment as a text box. Spans can only **nest inside `p`/`h*`** for local styling (bold, color change).

### Rule 2: No CSS Gradients — Solid Colors Only

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Correct: solid color */
background: #FF6B6B;

/* ✅ If multi-color stripes are needed, use flex children each with solid color */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint shape fill supports solid/gradient-fill, but pptxgenjs's `fill: { color: ... }` only maps to solid. Gradient fills would require a separate PowerPoint native gradient structure — not currently supported in the toolchain.

### Rule 3: Background / Border / Shadow on DIV Only — Not on Text Tags

```html
<!-- ❌ Wrong: <p> has background color -->
<p style="background: #FFD700; border-radius: 4px;">Key content</p>

<!-- ✅ Correct: outer div carries background/border, <p> handles text only -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key content</p>
</div>
```

**Why**: In PowerPoint, a shape (rectangle/rounded rectangle) and a text frame are two separate objects. HTML `<p>` translates to a text frame only — background/border/shadow belong on the **div wrapping the text**.

### Rule 4: No `background-image` on DIVs — Use `<img>` Tags

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Correct -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` only extracts image paths from `<img>` elements — it doesn't parse CSS `background-image` URLs.

---

## Path A HTML Template Skeleton

One independent HTML file per slide, with naturally isolated scope (avoids single-file deck CSS pollution).

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ match LAYOUT_WIDE */
    font-family: system-ui, -apple-system, sans-serif;
    background: #FEFEF9;                    /* solid color — no gradients */
    overflow: hidden;
  }
  /* DIV handles layout / background / border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* background on the div */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags handle only typography — no background/border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Header: outer div handles positioning, inner text tags handle content -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Headline as an assertion, not a topic word</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle supplementary note</p>
  </div>

  <!-- Content card: div carries background, h2/p carry text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Point One</h2>
    <p>Brief supporting copy</p>
  </div>

  <!-- List: use ul/li — not manual bullet characters -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First key point</li>
      <li>Second key point</li>
      <li>Third key point</li>
    </ul>
  </div>

  <!-- Image: use <img> tag — not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common Error Quick Reference

| Error message | Cause | Fix |
|---|---|---|
| `DIV element contains unwrapped text "XXX"` | Bare text directly in div | Wrap text in `<p>` or `<h1>`–`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Replace with solid color, or use flex children for segments |
| `Text element <p> has background` | `<p>` tag has background color | Wrap in outer `<div>` to carry background; `<p>` handles text only |
| `Background images on DIV elements are not supported` | div used background-image | Switch to `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content or decrease font size, or use `overflow: hidden` to clip |
| `HTML dimensions don't match presentation layout` | Body dimensions don't match presentation layout | Use `960pt × 540pt` body with `LAYOUT_WIDE`; or defineLayout for custom size |
| `Text box "XXX" ends too close to bottom edge` | Large-font `<p>` is less than 0.5 inch from body bottom | Move it up; leave enough bottom margin — PPT hides the bottom edge anyway |

---

## Basic Workflow (3 Steps to PPTX)

### Step 1: Write Each Slide as an Independent HTML File Following the Constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # all images referenced by <img> tags
    ├── chart1.png
    └── ...
```

### Step 2: Write build.js to Call `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // this skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch — matches HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and Check

- Open the exported PPTX in PowerPoint/Keynote
- Double-clicking any text should make it directly editable (if it's an image, Rule 1 was violated)
- Check for overflow: each slide should be within body bounds with nothing clipped

---

## This Path vs Alternatives (When to Choose What)

| Need | Choose |
|---|---|
| Colleagues will edit text in PPTX / sending to non-technical people to continue editing | **This path** (editable — HTML must follow the 4 constraints from the start) |
| Presenting only / archiving — no further editing | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage) — vector PDF |
| Visual priority (animation / web components / CSS gradients / complex SVG), non-editable is acceptable | **PDF** (same as above) — PDF is both high-fidelity and cross-platform; better than "image-based PPTX" |

**Never run html2pptx on visually-driven HTML** — real-world test: visually-driven HTML pass rate < 30%; the remaining per-slide remediation is slower than a full rewrite. This scenario should produce PDF, not force PPTX.

---

## Fallback: Visual Mockup Already Exists But User Insists on Editable PPTX

Occasionally you'll encounter this: you or the user has already written a visually-driven HTML (gradients, web components, complex SVG all used), PDF would be the right call, but the user explicitly says "No, must be editable PPTX."

**Don't just run `html2pptx` and hope it passes** — real-world test: visually-driven HTML pass rate <30% on html2pptx; the remaining 70% will error or render incorrectly. The correct fallback is:

### Step 1 · First Communicate the Constraints (Transparent)

One sentence to cover three things:

> "Your current HTML uses [list specifically: gradients / web components / complex SVG / ...]. Converting directly to editable PPTX will fail. I have two options:
> - A. **Export as PDF** (recommended) — 100% visual fidelity, recipients can view and print but can't edit text
> - B. **Rewrite an editable HTML version based on the visual mockup** (preserves color / layout / copy design decisions, but restructures HTML to follow the 4 hard constraints, **sacrificing** gradients, web components, complex SVG, and similar visual capabilities) → then exports editable PPTX
>
> Which do you prefer?"

Don't make Option B sound easy — explicitly state **what will be lost**. Let the user choose.

### Step 2 · If User Chooses B: AI Rewrites — Don't Ask the User to Rewrite Themselves

The principle here: **the user gives design intent; you translate it into a compliant implementation**. Don't ask the user to learn the 4 hard constraints and rewrite themselves.

Rewrite principles:
- **Preserve**: color system (primary / secondary / neutral), information hierarchy (headline / subtitle / body / footnote), core copy, layout skeleton (top-middle-bottom / two-column / grid), page rhythm
- **Downgrade**: CSS gradients → solid colors or flex segments; web components → block-level HTML; complex SVG → simplified `<img>` or solid-color geometry; shadows → removed or made very faint; custom fonts → lean toward system fonts
- **Rewrite**: bare text → wrapped in `<p>` / `<h*>`; `background-image` → `<img>` tags; `<p>` with background/border → moved to outer div

### Step 3 · Deliver a Before/After Comparison (Transparent)

After rewriting, give the user a before/after comparison so they know which visual details were simplified:

```
Original design → editable version adjustment
- Headline area purple gradient → primary color #5B3DE8 solid background
- Data card shadows → removed (replaced with 2pt border to distinguish)
- Complex SVG line chart → simplified to <img> PNG (screenshot from HTML)
- Hero web component animation → static first frame (web component can't translate)
```

### Step 4 · Export and Deliver Both Formats

- Editable HTML version → run `scripts/export_deck_pptx.mjs` to produce editable PPTX
- **Recommend also keeping** original visual mockup → run `scripts/export_deck_pdf.mjs` to produce high-fidelity PDF
- Deliver both formats: visual PDF + editable PPTX — each serves its own purpose

### When to Decline Option B Outright

In a few cases the rewrite cost is prohibitive — advise the user to abandon editable PPTX:
- HTML's core value is animation or interactivity (after rewriting, only a static first frame remains — 50%+ information loss)
- More than 30 pages — rewrite cost exceeds 2 hours
- Visual design deeply depends on precise SVG / custom filters (after rewriting, barely resembles the original)

In these cases tell the user: "Rewriting this deck is too costly — PDF is recommended over PPTX. If the recipient genuinely needs .pptx format, accept that visuals will be significantly simpler — or switch to PDF?"

---

## Why the 4 Constraints Are Physical Constraints, Not Bugs

These 4 rules aren't the `html2pptx.js` author cutting corners — they are **constraints of the PowerPoint file format (OOXML) itself**, projected onto HTML:

- Text in PPTX must exist in a text frame (`<a:txBody>`) — corresponds to block-level HTML elements
- PPTX shapes and text frames are two separate objects — a single element can't simultaneously have a background drawn and text written on it
- PPTX shape fill has limited gradient support (only certain preset gradients — arbitrary CSS angle gradients are not supported)
- PPTX picture objects must reference a real image file — not a CSS property

Understanding this: **don't expect the tool to get smarter** — the HTML writing style must adapt to PPTX format, not the other way around.
