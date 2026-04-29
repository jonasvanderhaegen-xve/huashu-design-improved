# Content Guidelines: Anti-AI Slop, Content Rules, Scale Specifications

The most common traps in AI design. This is a list of what not to do — more important than what to do, because AI slop is the default: if you don't actively avoid it, it will happen.

## Complete AI Slop Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradient (the typical smell of AI-generated web pages)
- Rainbow gradients in any direction
- Mesh gradient covering the entire background
- ✅ If you must use a gradient: keep it subtle, monochromatic, and intentional (e.g., on button hover)

**❌ Rounded cards + left border accent color**
```css
/* the signature pattern of AI-flavored cards */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This card pattern is ubiquitous in AI-generated dashboards. Want to create emphasis? Use more design-forward approaches: background color contrast, weight/size contrast, plain dividers — or don't use cards at all.

**❌ Emoji decoration**
Unless the brand itself uses emoji (like Notion or Slack), don't put emoji in UI. **Especially avoid:**
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → inside CTA buttons (a plain arrow is fine; an emoji arrow is not)

No icons? Use a real icon library (Lucide / Heroicons / Phosphor) or use a placeholder.

**❌ SVG-drawn imagery**
Don't try to draw people, scenes, devices, objects, or abstract art in SVG. AI-drawn SVG imagery is instantly identifiable as AI output — juvenile and cheap. **A gray rectangle with the text label "illustration placeholder 1200×800" is 100× better than a clumsy SVG hero illustration.**

The only acceptable SVG uses:
- True icons (16×16 to 32×32 scale)
- Geometric shapes as decorative elements
- Data viz charts

**❌ Excessive iconography**
Not every heading / feature / section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**❌ "Data slop"**
Fabricated stats as decoration:
- "10,000+ happy customers" (you don't even know if there are any)
- "99.9% uptime" (don't write it if you don't have real data)
- Decorative "metric cards" made of icon + number + label
- Fake data in mock tables dressed up to look impressive

If you don't have real data, use a placeholder or ask the user for it.

**❌ "Quote slop"**
Fabricated user testimonials or celebrity quotes decorating the page. Use a placeholder and ask the user for a real quote.

### Typography Traps

**❌ Avoid these overused fonts:**
- Inter (default for AI-generated web pages)
- Roboto
- Arial / Helvetica
- Pure system font stack
- Fraunces (AI found this and overused it)
- Space Grotesk (AI's current favorite)

**✅ Use distinctive display + body pairings.** Direction for inspiration:
- Serif display + sans-serif body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for weight animation in hero sections

Font resources:
- Lesser-known gems on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (siblings to Fraunces, Adobe Fonts)
- Don't invent font names from thin air

### Color Traps

**❌ Inventing colors from scratch**
Don't design an entire unfamiliar color palette from scratch. It usually lacks harmony.

**✅ Strategy:**
1. Have brand colors → use them; fill missing color tokens with oklch interpolation
2. No brand colors but have a reference → sample from a screenshot of the reference product
3. Starting from zero → pick a known color system (Radix Colors / Tailwind default palette / Anthropic brand) — don't tune it yourself

**Defining color with oklch** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter in same hue family */
  --primary-dark: oklch(0.45 0.20 25);  /* darker in same hue family */
}
```
oklch guarantees that adjusting lightness won't shift the hue — more reliable than hsl.

**❌ Casually inverting colors for dark mode**
It's not a simple color inversion. Good dark mode requires re-tuning saturation, contrast, and accent colors. If you don't want to do it properly, don't do it at all.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page tries to do bento. Unless your information structure actually suits bento, use a different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing page template is exhausted. If you want to be creative, actually be creative.

**❌ Every card in a card grid looks the same**
Asymmetric layouts, cards of varying sizes, some with images and some text-only, some spanning multiple columns — that's what real designers do.

## Content Rules

### 1. Don't add filler content

Every element must earn its place. Empty space is a design problem — solve it with **composition** (contrast, rhythm, whitespace), **not** by stuffing in content.

**Questions for identifying filler:**
- If you removed this content, would the design suffer? If the answer is "no", remove it.
- What real problem does this element solve? If the answer is "make the page less empty", delete it.
- Is this stat / quote / feature backed by real data? If not, don't write it.

"One thousand no's for every yes."

### 2. Ask before adding material

Think adding a section / page / extra paragraph would improve it? Ask the user first — don't add unilaterally.

Why:
- The user knows their audience better than you do
- Adding content has a cost — the user may not want it
- Adding content unilaterally violates the "junior designer checking in" relationship

### 3. Create a system up front

After exploring the design context, **state the system you intend to use out loud**, and let the user confirm:

```markdown
My design system:
- Color: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typefaces: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored background + white text; regular sections use white background
- Images: hero uses full-bleed photo; feature sections use placeholder until you provide them
- Maximum 2 background colors to avoid clutter

Confirm this direction and I'll start.
```

Wait for user confirmation before starting. This check-in prevents "realizing the direction was wrong halfway through".

## Scale Specifications

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28–36px
- Headings 60–120px
- Section title 80–160px
- Hero headline can use 180–240px large type
- Never use type smaller than 24px on slides

### Print Documents

- Body text minimum **10pt** (≈13.3px), ideal 11–12pt
- Headings 18–36pt
- Caption 8–9pt

### Web and Mobile

- Body text minimum **14px** (use 16px for accessibility / older users)
- Mobile body text **16px** (prevents iOS auto-zoom)
- Hit target (clickable elements) minimum **44×44px**
- Line height 1.5–1.7 (1.7–1.8 for CJK text)

### Contrast

- Body text vs background **at least 4.5:1** (WCAG AA)
- Large text vs background **at least 3:1**
- Check using Chrome DevTools accessibility tools

## CSS Power Tools

**Advanced CSS features** are a designer's best friend — use them freely:

### Typography

```css
/* natural headline wrapping — prevents a lone word on the last line */
h1, h2, h3 { text-wrap: balance; }

/* body text wrapping — avoids widows and orphans */
p { text-wrap: pretty; }

/* CJK typography: punctuation spacing and line-edge control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = maximum readability */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid to align card content */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual Effects

```css
/* styled scrollbar with design intent */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* glassmorphism (use sparingly) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View Transitions API for smooth page switching */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* :has() selector for easy conditional styles */
.card:has(img) { padding-top: 0; } /* cards with images: no top padding */

/* container queries for truly responsive components */
@container (min-width: 500px) { ... }

/* new color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Quick Reference: When in Doubt

- Want to add a gradient? → Probably don't
- Want to add an emoji? → Don't
- Want to add rounded corners + border-left accent to a card? → Don't; find another approach
- Want to draw a hero illustration in SVG? → Don't; use a placeholder
- Want to add a decorative quote? → Ask the user first if they have a real quote
- Want to add a row of icon features? → Ask whether icons are even needed; they probably aren't
- Using Inter? → Switch to something more distinctive
- Using a purple gradient? → Replace it with a color choice grounded in your context

**Whenever you think "adding this would look nicer" — that's usually a sign of AI slop.** Build the simplest version first; only add when the user asks.

---

## UX Writing Templates

### Error Messages

Formula: **what happened + why + how to fix**

| Context | ❌ Bad | ✅ Good |
|---|---|---|
| Expired session | "Error occurred" | "Session expired — sign in again to continue" |
| Invalid input | "Invalid email" | "Enter a valid email address (e.g. you@company.com)" |
| Permission denied | "403 Forbidden" | "You don't have access to this item. Ask an admin to add you." |
| Network failure | "Request failed" | "Couldn't connect — check your internet and try again" |
| File too large | "File error" | "This file is 24 MB. Maximum is 10 MB. Compress it or upload a smaller one." |
| Validation error | "Required field" | "Add a project name to continue" |

**Additional rules:**
- Blame-free language: "Couldn't save" not "You didn't save" / "Save failed" not "Your save failed"
- Specific: name the field, the limit, or the action that failed — never just "something went wrong"
- Preserve user's work: never clear a form on validation error
- Error text inline below the field + brief summary toast if multiple fields failed

### Destructive Action Labels

Name the object and count — never use vague verbs alone.

| ❌ Vague | ✅ Specific |
|---|---|
| "Delete" | "Delete 3 projects" |
| "Remove" | "Remove Jonas from team" |
| "Clear" | "Clear 14 days of history" |
| "Reset" | "Reset to defaults" |
| "Archive" | "Archive this conversation" |

Confirmation dialogs: always show the object being affected in the headline, not just the action.

```
Delete "Q4 Campaign"?
This will permanently remove the project and all 47 files inside it.
[Cancel]  [Delete project]  ← button label repeats the action
```

### Empty States

Every empty state needs: what's missing + why + what to do next.

| Context | ❌ | ✅ |
|---|---|---|
| No results | "No items found" | "No projects match 'invoice' — try a different search term" |
| First use | "No data" | "No activity yet — create your first project to get started" |
| Filtered to zero | "Nothing here" | "No tasks with 'High priority' — change filter or add a task" |
| Permission gap | "No content" | "You don't have access to any projects — ask an admin to add you" |

### Button Labels

Pattern: **verb + object** (not just noun, not just verb)

| ❌ | ✅ |
|---|---|
| "Submit" | "Save changes" / "Create account" / "Send message" |
| "OK" | "Got it" / "Continue" / "Done" |
| "Yes" | Repeat the action: "Yes, delete" / "Yes, sign out" |
| "Upload" (standalone) | "Upload photo" / "Add files" |
| "Continue" (vague) | "Continue to payment" / "Continue to review" |

**CTA on landing pages:** be specific about what happens next — "Start free trial" not "Get started", "See pricing" not "Learn more".

### Voice vs Tone

**Voice** is constant — the brand's personality doesn't change.  
**Tone** adapts to context — the same person sounds different in a success message vs a critical error.

| Context | Tone |
|---|---|
| Success / completion | Warm, brief. "Done! Your changes are saved." |
| Critical error / data loss risk | Calm, clear, no flourish. State facts. |
| Empty state / first use | Encouraging. Explain the value, not the absence. |
| Loading / progress | Reassuring. "Preparing your export…" not silent spinner. |
| Warning | Direct, non-alarming. "This will affect 3 other users." |

Avoid forced personality in high-stakes moments. "Oops! 🙈 Something broke" is fine for a 404. It's wrong for a payment failure.
