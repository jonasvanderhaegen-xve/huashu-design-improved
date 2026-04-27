# In-Depth Design Critique Guide

> Detailed reference for Phase 7. Provides scoring rubrics, context-specific emphasis, and a common issues checklist.

---

## Scoring Rubric Details

### 1. Philosophy Alignment

| Score | Standard |
|---|---|
| 9–10 | Design perfectly embodies the core spirit of the chosen philosophy; every detail has a philosophical rationale |
| 7–8 | Overall direction is correct, core characteristics are present, occasional detail drifts |
| 5–6 | Intent is visible, but execution mixes in elements from other styles — not pure enough |
| 3–4 | Surface-level imitation only; underlying philosophy not understood |
| 1–2 | Essentially unrelated to the chosen philosophy |

**Critique questions:**
- Does it use the signature techniques of that designer/studio?
- Do color, type, and layout fit within that philosophy's system?
- Are there self-contradicting elements? (e.g. choosing Kenya Hara then packing in content)

### 2. Visual Hierarchy

| Score | Standard |
|---|---|
| 9–10 | The viewer's eye flows naturally along the designer's intended path — zero-friction information access |
| 7–8 | Primary/secondary relationship is clear, with occasional 1–2 blurry hierarchy levels |
| 5–6 | Can distinguish headline from body, but intermediate levels are confused |
| 3–4 | Information is flat — no clear visual entry point |
| 1–2 | Chaotic — viewer doesn't know where to look first |

**Critique questions:**
- Is the type size contrast between headline and body sufficient? (at least 2.5×)
- Do color / weight / size establish 3–4 clear levels?
- Is whitespace guiding the eye?
- "Squint test": when you squint, is the hierarchy still clear?

### 3. Craft Quality

| Score | Standard |
|---|---|
| 9–10 | Pixel-perfect — alignment, spacing, color have no flaws |
| 7–8 | Overall precise, with 1–2 minor alignment or spacing issues |
| 5–6 | Roughly aligned, but inconsistent spacing and unsystematic color use |
| 3–4 | Obvious alignment errors, chaotic spacing, too many colors |
| 1–2 | Rough — looks like a draft |

**Critique questions:**
- Is there a consistent spacing system (e.g. 8pt grid)?
- Is spacing consistent across equivalent elements?
- Is the color count controlled? (typically no more than 3–4)
- Is the typeface family consistent? (typically no more than 2)
- Are edges and alignments precise?

### 4. Functionality

| Score | Standard |
|---|---|
| 9–10 | Every design element serves the goal — zero redundancy |
| 7–8 | Function-oriented is clear, with a small amount of removable decoration |
| 5–6 | Basically usable, but obvious decorative elements distract |
| 3–4 | Form over function — user has to work to find information |
| 1–2 | Completely buried in decoration — loses the ability to communicate information |

**Critique questions:**
- Would the design get worse if any element were removed? (If not, remove it)
- Is the CTA / key information in the most prominent position?
- Are there elements added "because they look good"?
- Does information density match the medium? (slides shouldn't be dense; PDF can be denser)

### 5. Originality

| Score | Standard |
|---|---|
| 9–10 | Refreshing — found a unique expression within the philosophical framework |
| 7–8 | Has its own ideas — not a simple template application |
| 5–6 | By-the-book — looks like a template |
| 3–4 | Heavy use of clichés (e.g. gradient sphere for AI) |
| 1–2 | Entirely template or asset-collage |

**Critique questions:**
- Are common clichés avoided? (see "Top 10 Common Design Issues" below)
- Is there personal expression while following the design philosophy?
- Are there "unexpected yet clearly right" design decisions?

---

## Context-Specific Critique Priorities

Different output types have different critical dimensions:

| Context | Most important | Second priority | Can be relaxed |
|---|---|---|---|
| Social media cover / feature image | Originality, visual hierarchy | Philosophy alignment | Functionality (single image, no interaction) |
| Infographic | Functionality, visual hierarchy | Craft quality | Originality (accuracy first) |
| Deck / Keynote | Visual hierarchy, functionality | Craft quality | Originality (clarity first) |
| PDF / whitepaper | Craft quality, functionality | Visual hierarchy | Originality (professionalism first) |
| Landing page / website | Functionality, visual hierarchy | Originality | — (full requirements) |
| App UI | Functionality, craft quality | Visual hierarchy | Philosophy alignment (usability first) |
| Lifestyle / editorial image | Originality, visual hierarchy | Philosophy alignment | Craft quality (atmosphere first) |

---

## Top 10 Common Design Issues

### 1. AI Tech Clichés
**Issue**: gradient orbs, digital rain, blue circuit boards, robot faces
**Why it's a problem**: audiences are visually fatigued by these; they can't distinguish you from everyone else
**Fix**: use abstract metaphors instead of literal symbols (e.g. metaphor for "dialogue" rather than a chat bubble icon)

### 2. Insufficient Type Scale Hierarchy
**Issue**: headline-to-body size gap too small (<2.5×)
**Why it's a problem**: viewers can't quickly locate key information
**Fix**: headline at least 3× body size (e.g. body 16px → headline 48–64px)

### 3. Too Many Colors
**Issue**: 5+ colors used with no hierarchy
**Why it's a problem**: visual chaos, weak brand identity
**Fix**: limit to 1 primary + 1 secondary + 1 accent + grays

### 4. Inconsistent Spacing
**Issue**: element spacing is arbitrary, no system
**Why it's a problem**: looks unprofessional, chaotic visual rhythm
**Fix**: establish an 8pt grid system (spacing only in 8/16/24/32/48/64px)

### 5. Insufficient Whitespace
**Issue**: every area of space is filled with content
**Why it's a problem**: crowded information causes reading fatigue, paradoxically reducing information transfer
**Fix**: whitespace should occupy at least 40% of total area (60%+ for minimalist styles)

### 6. Too Many Typefaces
**Issue**: 3+ typefaces used
**Why it's a problem**: visual noise, weakens unity
**Fix**: max 2 typefaces (1 for headings + 1 for body); create variation through weight and size

### 7. Inconsistent Alignment
**Issue**: some elements left-aligned, some centered, some right-aligned
**Why it's a problem**: destroys visual order
**Fix**: choose one alignment system (left-align recommended) and apply globally

### 8. Decoration Overpowers Content
**Issue**: background patterns / gradients / shadows compete with the primary content
**Why it's a problem**: priorities inverted — viewers come for information, not decoration
**Fix**: "If I removed this decoration, would the design get worse?" If not, remove it

### 9. Cyberpunk Neon Overuse
**Issue**: dark blue background (#0D1117) + neon glow effects
**Why it's a problem**: default taste violation (this skill's baseline), and it has become one of the biggest clichés — user can override with their brand
**Fix**: choose a more distinctive color system (reference the 20-style color systems)

### 10. Information Density Mismatched to Medium
**Issue**: full page of text in a slide / 10 elements crammed into a cover image
**Why it's a problem**: different media have different optimal information densities
**Fix:**
- Slides: 1 core idea per slide
- Cover image: 1 visual focal point
- Infographic: layered presentation
- PDF: can be denser, but needs clear navigation

---

## Critique Output Template

```
## Design Critique

**Overall score**: X.X/10 [Excellent (8+) / Good (6–7.9) / Needs work (4–5.9) / Fail (<4)]

**Dimension scores:**
- Philosophy Alignment: X/10 [one-line summary]
- Visual Hierarchy: X/10 [one-line summary]
- Craft Quality: X/10 [one-line summary]
- Functionality: X/10 [one-line summary]
- Originality: X/10 [one-line summary]

### Strengths (Keep)
- [Specific callout of what works well — use design language]

### Issues (Fix)
[Sorted by severity]

**1. [Issue name]** — ⚠️ Critical / ⚡ Important / 💡 Polish
- Current: [describe the state]
- Problem: [why this is a problem]
- Fix: [specific action, include values]

### Quick Wins
If you only have 5 minutes, prioritize these 3:
- [ ] [Most impactful fix]
- [ ] [Second most important fix]
- [ ] [Third most important fix]
```

---

**Version**: v1.0
**Updated**: 2026-02-13
