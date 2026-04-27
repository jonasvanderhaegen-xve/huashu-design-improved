# Slide Decks: HTML Slide Production Guide

Slides are a high-frequency design task. This document explains how to build good HTML slides — from architecture decisions and single-page design to the full PDF/PPTX export path.

**Capability coverage:**
- **HTML presentation (the base deliverable — always required by default)** → one HTML per slide + `assets/deck_index.html` aggregator, browser keyboard navigation, fullscreen presenter mode
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires HTML to follow 4 hard constraints)

> **⚠️ HTML is the foundation; PDF/PPTX are derivatives.** Regardless of final delivery format, you **must** first build the HTML aggregated presentation (`index.html` + `slides/*.html`). It is the "source" of the deck. PDF/PPTX are one-command snapshots exported from HTML.
>
> **Why HTML-first:**
> - Best for live presenting (projector / screen share — fullscreen in browser, keyboard navigation, no dependency on Keynote/PPT software)
> - During development, each slide can be double-clicked and validated independently — no need to re-run exports
> - Is the sole upstream for PDF/PPTX export (avoids the "discover a change needed after export, re-export" death loop)
> - Deliverables can be "HTML + PDF" or "HTML + PPTX" — the recipient uses whichever they prefer
>
> 2026-04-22 moxt brochure real test: after building 13 slides + index.html aggregator, `export_deck_pdf.mjs` exported the PDF in one command, zero changes needed. The HTML version itself was a browser-ready deliverable.

---

## 🛑 Confirm Delivery Format Before Starting (Hardest Checkpoint)

**This decision comes before "single-file vs multi-file."** 2026-04-20 stakeholder deck project real test: **not confirming delivery format before starting = 2–3 hours of rework.**

### Decision Tree (HTML-first Architecture)

Every delivery starts from the same HTML aggregated page (`index.html` + `slides/*.html`). Delivery format only determines **HTML writing constraints** and **export commands**:

```
[Always required by default] HTML aggregated presentation (index.html + slides/*.html)
   │
   ├── Browser-only presenting / local HTML archive   → done here; HTML has maximum visual freedom
   │
   ├── Also need PDF (printing / sharing / archiving) → run export_deck_pdf.mjs — one command
   │                                                      HTML writing unconstrained, no visual limits
   │
   └── Also need editable PPTX (colleagues will edit text) → write HTML with 4 hard constraints from line 1
                                                               run export_deck_pptx.mjs — one command
                                                               sacrifice gradients / web components / complex SVG
```

### Opening Script (copy and use)

> Regardless of final format — HTML, PDF, or PPTX — I'll always build a browser-navigable HTML aggregated version first (`index.html` with keyboard navigation). That's the permanent default base deliverable. On top of that I'll ask if you need additional PDF / PPTX snapshots.
>
> Which export format do you need?
> - **HTML only** (presenting / archiving) → visually unconstrained
> - **Also PDF** → same as above, plus one export command
> - **Also editable PPTX** (colleagues will edit text in PowerPoint) → I must write HTML from line 1 with 4 hard constraints, which sacrifices some visual capability (no gradients, no web components, no complex SVG).

### Why "Wanting PPTX Means 4 Hard Constraints from the Start"

Editable PPTX requires `html2pptx.js` to translate DOM elements one-by-one into PowerPoint objects. It needs **4 hard constraints:**

1. Body fixed at 960pt × 540pt (matches `LAYOUT_WIDE`, 13.333″ × 7.5″ — not 1920×1080px)
2. All text wrapped in `<p>`/`<h1>`–`<h6>` (no text directly in divs, no `<span>` as primary text containers)
3. `<p>`/`<h*>` tags themselves cannot have background/border/shadow (put those on an outer div)
4. `<div>` cannot use `background-image` (use `<img>` tags)
5. No CSS gradients, no web components, no complex SVG decorations

**This skill's default HTML has maximum visual freedom** — heavy use of spans, nested flex, complex SVG, web components (like `<deck-stage>`), CSS gradients — **almost none of it passes html2pptx constraints natively** (real test: visually-driven HTML submitted directly to html2pptx, pass rate < 30%).

### Cost Comparison: Two Real Paths (2026-04-20 real incident)

| Path | Approach | Result | Cost |
|---|---|---|---|
| ❌ **Write HTML freely, patch PPTX later** | Single-file deck-stage + heavy SVG/span decoration | Only two options left for editable PPTX:<br>A. Hand-write hundreds of lines of hardcoded pptxgenjs coordinates<br>B. Rewrite 17 pages of HTML in Path A format | 2–3 hours rework, and the hand-written version has **permanent maintenance debt** (change one word in HTML → manually sync PPTX again) |
| ✅ **Follow Path A constraints from step one** | Separate HTML per slide + 4 hard constraints + 960×540pt | One command exports 100% editable PPTX; also works as browser fullscreen presentation (Path A HTML is standard browser-playable HTML) | 5 extra minutes thinking "how to wrap text in `<p>`" — zero rework |

### Handling Mixed Delivery

User says "I want HTML presentation **and** editable PPTX" — **this isn't mixed**, PPTX requirements cover HTML requirements. HTML written to Path A constraints can natively present in a browser (just add `deck_index.html` as aggregator). **No extra cost.**

User says "I want PPTX **and** animations / web components" — **this is a genuine conflict**. Tell the user: editable PPTX means sacrificing these visual capabilities. Let them choose — don't silently build a hand-written pptxgenjs solution (it becomes permanent maintenance debt).

### Emergency Recovery: PPTX Discovered After HTML Is Written

Rare situation: HTML is already done before PPTX requirement surfaces. Follow the **fallback flow** (full explanation in `references/editable-pptx.md`, "Fallback: Visual mockup already exists but user insists on editable PPTX"):

1. **Preferred: produce PDF** (100% visual fidelity, cross-platform, recipient can view and print) — if the actual need is "presenting / archiving," PDF is the optimal deliverable
2. **Secondary: AI rewrites an editable HTML version based on the visual mockup** → exports editable PPTX — preserves color / layout / copy design decisions, sacrifices gradients, web components, complex SVG
3. **Not recommended: hand-write pptxgenjs reconstruction** — positions, fonts, alignment all need manual calibration; high maintenance cost; any HTML text change must be re-synced manually

Always present the options to the user and let them decide. **Never reach for hand-written pptxgenjs as a first response** — it's the last-resort fallback.

---

## 🛑 Before Batch Production: Do a 2-Slide Showcase to Lock the Visual Grammar

**Any deck of 5+ slides: never write straight from slide 1 to the end.** Correct order, validated in the 2026-04-22 moxt brochure real test:

1. Choose **2 visually distinct slide types** and build them as a showcase first (e.g. "cover" + "emotion/quote page", or "cover" + "product feature page")
2. Screenshot and confirm the visual grammar with the user (masthead / type / color / spacing / structure / bilingual ratio)
3. Once direction is approved, batch-build the remaining N–2 slides — each one reuses the established grammar
4. When all are done, assemble the HTML aggregator + PDF/PPTX derivatives

**Why**: writing all 13 slides to completion → user says "direction is off" = 13 rounds of rework. Build 2 showcase slides → direction is off = 2 rounds of rework. Once the visual grammar is established, the decision space for the remaining N slides collapses dramatically — only "how to fit the content in" remains.

**How to choose showcase slides**: pick the two pages with the most visually distinct structure. If those two pass, all intermediate cases will too.

| Deck type | Recommended showcase pair |
|---|---|
| B2B brochure / product launch | Cover + content page (philosophy/emotional page) |
| Brand launch | Cover + product features page |
| Data report | Large data visualization page + analysis conclusion page |
| Tutorial / course material | Chapter section page + specific knowledge content page |

---

## Publication-Style Grammar Template (moxt-tested, reusable)

Suitable for B2B brochures / product launches / long report decks. Each slide reuses this structure = 13 slides visually consistent, zero rework.

### Per-Slide Skeleton

```
┌─ masthead (top strip + rule line) ────────────────────────────────┐
│  [logo 22-28px] · A Product Brochure               Issue · Date · URL │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ── kicker (brand-color short rule + uppercase label)             │
│  CHAPTER XX · SECTION NAME                                        │
│                                                                    │
│  H1 (Noto Serif SC 900)                                           │
│  Key word in brand primary color                                  │
│                                                                    │
│  English subtitle (Lora italic, sub-headline)                     │
│  ─────────────────────────────────────────                        │
│                                                                    │
│  [Content: 60/40 two-column / 2×2 grid / list]                   │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│ section name                                             XX / total │
└────────────────────────────────────────────────────────────────────┘
```

### Style Conventions (copy directly)

- **H1**: Noto Serif SC 900, 80–140px depending on content volume; key words in brand primary color only (don't color the whole line)
- **English subtitle**: Lora italic 26–46px; brand signature words (e.g. "AI team") bold + primary color italic
- **Body**: Noto Serif SC 17–21px, line-height 1.75–1.85
- **Accent highlights**: bold key words in brand primary color in body text; max 3 per slide (too many and they lose their anchoring function)
- **Background**: warm cream `#FAFAFA` + very faint radial-gradient noise (`rgba(33,33,33,0.015)`) for paper texture

### Visual Lead Element Must Vary

13 slides all "text + one screenshot" is too monotonous. **Rotate the visual lead type per slide:**

| Visual type | Suitable section |
|---|---|
| Cover typography (large type + masthead + pillars) | Title slide / chapter opener |
| Single character portrait (oversized single figure) | Introducing a single concept / character |
| Group portrait / avatar cards side by side | Team / user case studies |
| Timeline card progression | Showing "long-term relationship" / "evolution" |
| Knowledge graph / connection node map | Showing "collaboration" / "flow" |
| Before/After comparison cards + center arrow | Showing "change" / "difference" |
| Product UI screenshot + outlined device frame | Specific feature showcase |
| Large-quote block (half-page large text) | Emotional / question / citation page |
| Real headshot + quote card (2×2 or 1×4) | User testimonial / usage scenario |
| Large-text back cover + URL oval button | CTA / ending |

---

## Common Pitfalls (moxt Real-World Summary)

### 1. Emoji Don't Render in Chromium / Playwright Export

Chromium doesn't include a color emoji font by default. `page.pdf()` or `page.screenshot()` renders emoji as empty boxes.

**Fix**: use Unicode text symbols (`✦` `✓` `✕` `→` `·` `—`) instead, or switch to plain text ("Email · 23" rather than "📧 23 emails").

### 2. `export_deck_pdf.mjs` Throws `Cannot find package 'playwright'`

Cause: ESM module resolution walks up from the script's location for `node_modules`. The script lives in `~/.claude/skills/huashu-design/scripts/` — no dependencies there.

**Fix**: copy the script to the deck project directory (e.g. `brochure/build-pdf.mjs`), run `npm install playwright pdf-lib` in the project root, then `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Google Fonts Not Loaded Before Screenshot → Chinese Shows as System Default

Playwright screenshot/PDF needs at least `wait-for-timeout=3500` to let webfonts download and paint. Alternatively, self-host fonts in `shared/fonts/` to reduce network dependency.

### 4. Information Density Imbalance: Content Slides Packed Too Full

The moxt philosophy page's first version had 2×2 = 4 paragraphs + 3 bottom beliefs = 7 content blocks — crowded and repetitive. Reducing to 1×3 = 3 paragraphs immediately restored breathing room.

**Fix**: limit each slide to "1 core message + 3–4 supporting points + 1 visual lead." If it exceeds this, split to a new slide. **Less is more** — the audience sees a slide for 10 seconds; 1 memorable point is more memorable than 4.

---

## 🛑 Decide Architecture First: Single-File or Multi-File?

**This choice is the first step of slide production. Getting it wrong means recurring pitfalls. Read this section before touching code.**

### Architecture Comparison

| Dimension | Single-file + `deck_stage.js` | **Multi-file + `deck_index.html` aggregator** |
|---|---|---|
| Code structure | One HTML, all slides are `<section>` | Separate HTML per slide; `index.html` assembles via iframe |
| CSS scope | ❌ Global — one slide's styles can affect all slides | ✅ Naturally isolated — each iframe is its own scope |
| Validation granularity | ❌ Must use JS goTo to navigate to a slide | ✅ Double-click any slide file to see it in browser |
| Parallel development | ❌ One file — multi-agent edits conflict | ✅ Multi-agent can build different slides in parallel — zero-conflict merge |
| Debugging | ❌ One CSS error breaks the whole deck | ✅ One slide error only affects itself |
| In-deck interactivity | ✅ Shared state across slides is simple | Cross-iframe requires postMessage |
| Print PDF | ✅ Built-in | ✅ Aggregator handles `beforeprint` across iframes |
| Keyboard navigation | ✅ Built-in | ✅ Built into the aggregator |

### Which to Choose? (Decision Tree)

```
│ Question: how many slides?
├── ≤10 slides, needs in-deck animation or cross-slide state, pitch deck → single-file
└── ≥10 slides, academic talk, course material, long deck, multi-agent parallel → multi-file (recommended)
```

**Default to multi-file.** It's not the "backup option" — it's **the primary path for long decks and team collaboration**. Every advantage of single-file (keyboard navigation, printing, scaling) is also available in multi-file, while multi-file's scope isolation and per-slide validation are advantages single-file can't recover.

### Why Is This Rule So Hard? (Real Incident Record)

Single-file architecture hit four consecutive bugs in an AI psychology lecture deck:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all slides to render stacked simultaneously.
2. **Shadow DOM slot rules suppressed by outer CSS**: `::slotted(section) { display: none }` couldn't block outer rule overrides — sections refused to hide.
3. **localStorage + hash navigation race condition**: after refresh, navigation stopped at the position stored in localStorage rather than jumping to the hash position.
4. **High validation cost**: had to `page.evaluate(d => d.goTo(n))` to screenshot a specific slide, twice as slow as directly `goto(file://.../slides/05-X.html)`, and frequently threw errors.

All root causes traced to **a single global namespace** — multi-file architecture eliminates these problems at the physical level.

---

## Path A (Default): Multi-File Architecture

### Directory Structure

```
MyDeck/
├── index.html              # copied from assets/deck_index.html, edit MANIFEST
├── shared/
│   ├── tokens.css          # shared design tokens (palette / type scale / chrome)
│   └── fonts.html          # <link> for Google Fonts (included by each slide)
└── slides/
    ├── 01-cover.html       # each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Per-Slide Template Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this slide. Any class name here won't affect other slides. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (body width/height locked by tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints:**
- `<body>` is the canvas — lay out directly inside it. Don't wrap in `<section>` or other wrappers.
- `width: 1920px; height: 1080px` locked by `shared/tokens.css`'s `body` rule.
- Include `shared/tokens.css` for shared design tokens (palette, type scale, page-header/footer chrome).
- Font `<link>` written per slide (fonts are cheap to import independently, and ensures each slide opens standalone).

### Aggregator: `deck_index.html`

**Copy directly from `assets/deck_index.html`.** The only thing you need to change is the `window.DECK_MANIFEST` array — list all slide filenames in order with human-readable labels:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

The aggregator has built-in: keyboard navigation (←/→/Home/End/number keys/P to print), scale + letterbox, bottom-right slide counter, localStorage memory, hash deep-linking, print mode (loops through iframes to output PDF page by page).

### Per-Slide Validation (the killer advantage of multi-file)

Every slide is an independent HTML file. **When one is done, double-click to open it in the browser:**

```bash
open slides/05-personas.html
```

Playwright screenshots go directly to `goto(file://.../slides/05-personas.html)` — no JS navigation needed, no interference from other slides' CSS. This makes "change a little, verify immediately" nearly zero-cost.

### Parallel Development

Assign each slide to a different agent and run them simultaneously — HTML files are independent, no merge conflicts. On a long deck, parallel production can compress build time to 1/N.

### What Belongs in `shared/tokens.css`

Only **genuinely cross-slide** content:

- CSS variables (palette, type scale, spacing scale)
- `body { width: 1920px; height: 1080px; }` canvas lock
- `.page-header` / `.page-footer` chrome that looks identical on every slide

**Don't** put single-slide layout classes here — that degrades back to the single-file global pollution problem.

---

## Path B (Small Decks): Single-File + `deck_stage.js`

Use for ≤10 slides, needing cross-slide shared state (e.g. a React tweaks panel that controls all slides), or pitch deck demos that need to be extremely compact.

### Basic Usage

1. Read content from `assets/deck_stage.js`, embed in HTML `<script>` (or `<script src="deck_stage.js">`)
2. Wrap slides in `<deck-stage>` in body
3. 🛑 **Script tag must be placed after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script Position Hard Constraint (2026-04-20 Real Incident)

**Do not put `<script src="deck_stage.js">` in `<head>`.** Even if it defines `customElements` in `<head>`, the parser triggers `connectedCallback` when it encounters the `<deck-stage>` opening tag — at that point, the child `<section>` elements haven't been parsed yet. `_collectSlides()` gets an empty array, counter shows `1 / 0`, all slides render stacked simultaneously.

**Three compliant options (choose any one):**

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also valid: script in head with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also valid: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` already has a `DOMContentLoaded` deferred-collection defense built in — it won't completely break even with the script in `<head>`. But `defer` or placing it at the bottom of body is still the cleaner approach, avoiding dependence on the defense branch.

### ⚠️ Single-File Architecture CSS Trap (Read This)

The most common pitfall in single-file architecture — **`display` property hijacked by per-slide styles.**

Common wrong approach 1 (writing `display: flex` directly on section):

```css
/* ❌ External CSS specificity 2 overrides shadow DOM's ::slotted(section){display:none} (also specificity 2) */
deck-stage > section {
  display: flex;            /* All slides render simultaneously, stacked! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common wrong approach 2 (section has a higher-specificity class):

```css
.emotion-slide { display: grid; }   /* specificity: 10 — even worse */
```

Both cause **all slides to render simultaneously, stacked** — counter may show `1 / 10` as if normal, but visually slide 1 is layered over slide 2 over slide 3.

### ✅ Starter CSS (Copy Directly — No Pitfalls)

**Section itself** only handles "visible / hidden"; **layout (flex/grid etc.) goes on `.active`:**

```css
/* Section defines only non-display universal styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Never write display here! */
}

/* Lock "non-active = hidden" — double defense with specificity + !important */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Active slide is where you write the required display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all slides must show — override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **put single-slide flex/grid on an inner wrapper `<div>`**, with the section itself only ever switching between `display: block/none`. This is the cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom Dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 portrait -->
</deck-stage>
```

---

## Slide Labels

Both `deck_stage` and `deck_index` label each slide (shown in the counter). Give them **more meaningful** labels:

**Multi-file**: write `{ file, label: "04 Problem Statement" }` in `MANIFEST`
**Single-file**: add `<section data-screen-label="04 Problem Statement">` to the section

**Key: slide numbers start at 1, not 0.**

When the user says "slide 5," they mean the 5th slide — never array index `[4]`. Humans don't talk in zero-indexed terms.

---

## Speaker Notes

**Off by default.** Only add when the user explicitly asks.

Adding speaker notes lets you reduce the text on each slide to a minimum, focusing on impactful visuals — notes carry the full script.

### Format

**Multi-file**: write in `index.html`'s `<head>`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single-file**: same location.

### Notes Writing Principles

- **Complete**: not an outline — the actual words to be spoken
- **Conversational**: natural spoken language, not written prose
- **Indexed**: array item N corresponds to slide N
- **Length**: 200–400 words is ideal
- **Emotional flow**: mark emphasis, pauses, and stress points

---

## Slide Design Patterns

### 1. Establish a System First (Required)

After exploring design context, **state the system you'll use before building:**

```markdown
Deck system:
- Background: max 2 colors (90% white + 10% dark section dividers)
- Typography: display in Instrument Serif, body in Geist Sans
- Rhythm: section dividers use full-bleed color + white text; standard slides white background
- Images: hero slides use full-bleed photos; data slides use charts

Building to this system. Tell me if anything looks off.
```

Get user confirmation before proceeding.

### 2. Common Slide Layouts

- **Title slide**: solid background + oversized headline + subtitle + author / date
- **Section divider**: colored background + section number + section title
- **Content slide**: white background + title + 1–3 bullet points
- **Data slide**: title + large chart / number + brief caption
- **Image slide**: full-bleed photo + small bottom caption
- **Quote slide**: negative space + oversized quote + attribution
- **Two-column**: left-right comparison (vs / before-after / problem-solution)

Use at most 4–5 layouts in a single deck.

### 3. Scale (Emphasized Again)

- Body minimum **24px**, ideal 28–36px
- Headings **60–120px**
- Hero text **180–240px**
- Slides are viewed from 10 meters away — text must be large enough

### 4. Visual Rhythm

Decks need **intentional variety:**

- Color rhythm: mostly white + occasional colored section dividers + occasional dark segments
- Density rhythm: some text-heavy slides + some image-heavy slides + some quote-and-space slides
- Type scale rhythm: standard headings + occasional oversized hero text

**Don't make every slide look the same** — that's a template, not design.

### 5. Breathing Room (Required Reading for Information-Dense Slides)

**The most common beginner mistake**: cramming every piece of information onto one slide.

Information density ≠ effective information delivery. Academic / lecture decks especially need restraint:

- List / matrix slides: don't draw all N elements at the same size. Use **hierarchy** — the 5 items relevant today enlarged as leads; the remaining 16 shrunk as background hints.
- Large-number slides: the number itself is the visual lead. Surrounding captions should be no more than 3 lines; otherwise the audience's eyes jump back and forth.
- Quote slides: leave whitespace between the quote and the attribution — don't let them touch.

Test yourself: "Is the data the lead?" and "Is the text crowded?" Edit until the whitespace makes you slightly uncomfortable.

---

## Print to PDF

**Multi-file**: `deck_index.html` already handles the `beforeprint` event and outputs PDF page by page.

**Single-file**: `deck_stage.js` does the same.

Print styles are already written — no additional `@media print` CSS needed.

---

## Export as PPTX / PDF (Self-Serve Scripts)

HTML-first is the primary deliverable. But users frequently need PPTX/PDF. Two general-purpose scripts in `scripts/` work with **any multi-file deck:**

### `export_deck_pdf.mjs` — Export Vector PDF (Multi-File Architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Characteristics:**
- Text **remains vector** (copyable, searchable)
- 100% visual fidelity (Playwright's built-in Chromium renders then prints)
- **No HTML changes required**
- Each slide gets its own `page.pdf()` call, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limitation**: PDF text is not editable — to change content, edit the HTML and re-export.

### `export_deck_stage_pdf.mjs` — For Single-File deck-stage Architecture Only ⚠️

**When to use**: the deck is a single HTML file with `<deck-stage>` web component wrapping N `<section>` elements (Path B architecture). The "one HTML per `page.pdf()`" approach of `export_deck_pdf.mjs` doesn't work here.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why `export_deck_pdf.mjs` can't be reused** (2026-04-20 real incident):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS has `::slotted(section) { display: none }` (only the active slide `display: block`). Even `@media print { deck-stage > section { display: block !important } }` in light DOM can't override it — `page.pdf()` triggers the print media query and Chromium's final render has only the active slide. **Entire PDF = 1 page** (the current active slide repeated).

2. **Looping goto per slide still only outputs 1 page**: the intuitive solution "navigate to each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — the `display: block` rule in print CSS is overridden within the shadow DOM, so the final render is always the first section in the list (not the one you navigated to). 17 loop iterations = 17 copies of slide P01.

3. **Absolute-positioned children overflow into the next page**: even if all sections render, if section itself is `position: static`, its absolute-positioned `cover-footer` / `slide-footer` children will position relative to the initial containing block. When print forces section height to 1080px, the absolute footer may get pushed onto the next page (PDF has more pages than sections — the extra pages contain only orphaned footers).

**Fix strategy** (implemented in the script):

```js
// After opening HTML, use page.evaluate to move sections out of deck-stage slot
// and attach them directly to a plain div under body, with inlined styles ensuring
// position:relative + fixed dimensions
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Last slide: disable page break to prevent trailing blank page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works:**
- Moving sections from shadow DOM slot to a plain div in light DOM completely bypasses `::slotted(section) { display: none }`
- Inline `position: relative` ensures absolute-positioned children position relative to the section, no overflow
- `page-break-after: always` makes the browser print each section as its own page
- Last child has no page break — prevents trailing blank page

**Note on `mdls -name kMDItemNumberOfPages` verification**: macOS Spotlight metadata is cached. After rewriting a PDF, run `mdimport file.pdf` to force refresh; otherwise it shows the old page count. Use `pdfinfo` or `pdftoppm` to count pages reliably.

---

### `export_deck_pptx.mjs` — Export Editable PPTX

```bash
# Single mode: text boxes are natively editable (fonts fall back to system fonts)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads `computedStyle` element-by-element and translates DOM into PowerPoint objects (text frames / shapes / pictures). Text becomes real text boxes — double-click to edit in PowerPoint.

**Hard constraints** (HTML must satisfy these, or the slide is skipped — full details in `references/editable-pptx.md`):
- All text must be in `<p>`/`<h1>`–`<h6>`/`<ul>`/`<ol>` (no bare text in divs)
- `<p>`/`<h*>` tags themselves cannot have background/border/shadow (put on outer div)
- No `::before`/`::after` decorative text (pseudo-element content can't be extracted)
- Inline elements (span/em/strong) cannot have margin
- No CSS gradients (not renderable)
- No `background-image` on divs (use `<img>`)

The script includes a built-in **auto-preprocessor** that wraps "bare text in leaf divs" into `<p>` (class preserved). This resolves the most common violation. But other violations (border on `<p>`, margin on `<span>`, etc.) still require compliant HTML at the source.

**Font fallback caveat:**
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote uses local fonts to render
- When they differ, expect **overflow or misalignment** — visually check every slide
- Recommend installing the fonts used in HTML on the target machine, or falling back to `system-ui`

**Visual-priority scenarios should not use this path** → use `export_deck_pdf.mjs` instead. PDF has 100% visual fidelity, vector text, cross-platform compatibility, and searchable text — the true destination for visual-priority decks, not a "can't-edit compromise."

### Making HTML Export-Friendly from the Start

The most reliable decks: **write HTML with the 4 editable constraints from the very first line.** Then `export_deck_pptx.mjs` can pass every slide. The additional cost is minimal:

```html
<!-- ❌ Bad -->
<div class="title">Key Finding</div>

<!-- ✅ Good (p wrapper, class inherited) -->
<p class="title">Key Finding</p>

<!-- ❌ Bad (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### Which to Choose

| Situation | Recommended |
|---|---|
| Submission to organizers / archiving | **PDF** (universal, high-fidelity, searchable text) |
| Sharing with collaborators who will tweak text | **Editable PPTX** (accept font fallback) |
| Live presenting, no content changes | **PDF** (vector fidelity, cross-platform) |
| HTML is the primary presentation medium | Present directly in browser; export is just a backup |

## Deep Path for Editable PPTX (Long-Term Projects Only)

If your deck will be maintained long-term, revised repeatedly, and collaboratively edited — start writing HTML to html2pptx constraints from the beginning, so `export_deck_pptx.mjs` can pass every slide. See `references/editable-pptx.md` (4 hard constraints + HTML template + common error reference + fallback flow for existing visual mockups).

---

## Common Issues

**Multi-file: iframe page won't open / white screen**
→ Check that `MANIFEST`'s `file` paths are correct relative to `index.html`. Use browser DevTools to verify the iframe `src` is directly accessible.

**Multi-file: one slide's styles conflicting with another**
→ Impossible (iframe isolation). If it looks like a conflict, it's a cache issue — force-refresh with Cmd+Shift+R.

**Single-file: multiple slides rendering simultaneously, stacked**
→ CSS specificity issue. See "Single-File Architecture CSS Trap" above.

**Single-file: scaling looks wrong**
→ Check that all slides are directly under `<deck-stage>` as `<section>` elements. No intermediate `<div>` wrapper.

**Single-file: want to jump to a specific slide**
→ Add hash to URL: `index.html#slide-5` jumps to slide 5.

**Both architectures: text positions inconsistent across screens**
→ Use fixed dimensions (1920×1080) and `px` units — no `vw`/`vh` or `%`. Scaling is handled uniformly.

---

## Validation Checklist (Run After Finishing the Deck)

1. [ ] Open `index.html` (or main HTML) directly in browser — first slide has no broken images, fonts loaded
2. [ ] Press → to go through every slide — no blank slides, no layout shifts
3. [ ] Press P for print preview — each page is exactly one A4 (or 1920×1080) with no clipping
4. [ ] Randomly pick 3 slides, force-refresh with Cmd+Shift+R — localStorage memory works correctly
5. [ ] Playwright batch screenshots (multi-file: loop `slides/*.html`; single-file: use goTo navigation) — human visual review of all slides
6. [ ] Search for `TODO` / `placeholder` residuals — confirm all cleared
