# Design Register: Brand vs Product

> Two fundamentally different modes of design. Choosing the wrong register is the most common source of "it looks wrong but I can't say why." Read this before making decisions about color palette depth, typography strategy, or component personality for any design involving a specific brand or product.

---

## The Register Decision

**Brand register**: the design IS the message. Visual expression serves recognition and emotional impression. The brand's personality should be unmistakable without the logo.

**Product register**: the design serves the task. The UI disappears into the user's work. A great product UI is invisible — the user thinks about what they're doing, not how the interface looks.

**How to decide:**

| Signal | Register |
|---|---|
| One-time impression: launch film, landing page, pitch deck, brand campaign | **Brand** |
| Ongoing work tool: app, dashboard, form-heavy flow, SaaS product | **Product** |
| Physical product showcase | **Brand** |
| Digital tool with repeated daily use | **Product** |
| Marketing material for an audience who doesn't know the brand yet | **Brand** |
| Internal tool, admin panel, developer dashboard | **Product** |
| User is experiencing, not accomplishing | **Brand** |
| User is accomplishing, not experiencing | **Product** |

When in doubt: ask whether the user is meant to notice the design. Brand = yes. Product = no.

---

## Color Strategy by Register

### Brand — Committed palette

Personality over neutrality. One or two dominant non-neutral colors. Braveness is rewarded. The palette should communicate something about the brand's character before the user reads a word.

```css
/* Brand: Anthropic */
:root {
  --bg: #e8e3d9;          /* warm off-white — distinctive, not generic */
  --text: #1a1a1a;
  --accent: #c96442;      /* terracotta — warm, not corporate */
  --accent-2: #4a3728;    /* deep brown — depth */
}
```

**Test:** cover the logo. Can a knowledgeable user still identify the brand? If not, the color isn't committed enough.

**Failure mode:** neutral palette + "unique" typography. Fonts can't carry brand identity alone.

### Product — Restrained palette

Neutrals dominate. Non-neutral colors only for semantic meaning. Color carries information, not personality.

```css
/* Product: functional SaaS */
:root {
  --bg: #ffffff;
  --surface: #f7f7f8;
  --border: #e4e4e7;
  --text: #18181b;
  --text-subtle: #71717a;

  /* semantic — these are not brand colors, they're state colors */
  --color-primary: oklch(0.55 0.18 250);   /* primary action only */
  --color-error: oklch(0.50 0.22 25);
  --color-warning: oklch(0.70 0.18 75);
  --color-success: oklch(0.55 0.16 145);
}
```

**Test:** replace every instance of `--color-primary` with a different hue. Does the product still feel like itself? If yes, the palette is correctly restrained — it's functional, not expressive.

**Failure mode:** injecting brand personality into a work tool. Users who use your product 8 hours a day don't want to feel the personality — they want to finish the task.

---

## Typography Strategy by Register

### Brand — Display pairing

The display font embodies the brand's character. It should feel intentional and slightly unexpected. Pair an expressive headline face with a functional body face.

**Decision process:**
1. Write 3 words that describe the brand's character (e.g. "warm, precise, trustworthy")
2. Map those words to a physical object or material (e.g. "aged paper, copper type, hand-crafted")
3. Find typefaces that evoke that material — not that spell out the words literally

**Good pairings by brand character:**

| Character | Display | Body |
|---|---|---|
| Editorial, prestigious | Freight Display / Cormorant | GT America / Neue Haas |
| Technical, precise | Söhne Mono / JetBrains Mono | Inter (legitimate here — product-adjacent) |
| Warm, human | Recoleta / Fraunces | DM Sans / Instrument Sans |
| Geometric, systematic | Acumin / Aktiv Grotesk | Same family, lighter weight |
| Expressive, cultural | Bricolage Grotesque / Syne | Geist / IBM Plex Sans |

**Avoid as brand display:** Inter, Roboto, Arial, Space Grotesk, Fraunces-as-default. These have been used by so many AI-generated designs that they signal "no typographic decision was made here."

### Product — Single functional family

One typeface family for all levels. Differentiate via weight and size only — not family switching. The typeface should feel neutral; the content is the personality.

**Legitimate choices for product typography:**
- Inter (yes — it became a cliché in brand contexts, but it's a legitimate tool for a product)
- Geist (clean, modern, slightly distinctive)
- DM Sans (warmer than Inter, still functional)
- IBM Plex Sans (technical products — carries a sense of reliability)
- System font stack (admin tools, developer tools — signals "this is infrastructure, not decoration")

**Hierarchy in product:**
```css
/* All in one family — differentiate by size + weight only */
--text-hero:    clamp(2rem, 5vw, 3.5rem); font-weight: 700;
--text-heading: 1.5rem;  font-weight: 600;
--text-body:    1rem;    font-weight: 400;
--text-small:   0.875rem; font-weight: 400;
--text-label:   0.75rem; font-weight: 500; letter-spacing: 0.05em;
```

---

## Component Personality by Register

### Brand — Bespoke shapes, intentional details

Components can break conventions if it serves the brand language. Unusual border radii, unexpected transitions, unconventional layout. Every component detail should trace back to the brand character.

- Border radius: whatever the brand calls for (sharp for tech precision, generous curves for warmth, asymmetric for energy)
- Buttons: can have distinctive shapes, but must still look interactive
- Cards: can have non-standard proportions, unexpected overlaps, editorial asymmetry

### Product — Semantic clarity, platform conventions

Components should be instantly recognizable as what they are. No ambiguity. Interactive elements look interactive. Non-interactive elements don't.

- Border radius: 4–6px for inputs/cards (enough to soften, not enough to be expressive)
- Buttons: standard elevated appearance with clear hover state
- Tables and lists: lean default, with density options for power users

**Product component test:** can a user from a competing product start using this with zero onboarding? If not — simplify.

---

## Mixed-Register Designs

Some designs need both: a brand-forward landing page that links to a product-register app. Treat them as separate surfaces:

- Landing page, marketing pages, onboarding splash: brand register
- Dashboard, settings, forms, table views: product register
- Transition: the brand colors appear in product as the primary action color only — not as a dominant surface color

**Common mistake:** using the brand's expressive color palette on the product's data tables and forms. The result looks unprofessional and creates visual noise where users need calm focus.

---

## Quick-Check Before Starting

Before making any color or typography decision on a new design task:

1. What register? (brand / product / mixed surface)
2. If brand: what 3 words describe the personality? What font evokes those words?
3. If product: what's the primary task? Does every color choice serve that task or just express personality?
4. Is the palette committed (brand) or restrained (product)?
5. Is the typography doing brand work (display pairing) or task work (single family)?
