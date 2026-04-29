# Cognitive Load in Design

> Use when designing multi-step flows, onboarding screens, dashboards, or information-dense slides. Applies to both interactive prototypes and presentation design.

---

## Three Types of Load (Sweller)

| Type | Definition | Controllable? |
|---|---|---|
| **Intrinsic** | Inherent complexity of the content itself | No — simplifying it means changing the information |
| **Extraneous** | How the design presents the content | **Yes — always reducible** |
| **Germane** | Processing that builds understanding (connecting ideas, pattern recognition) | Desirable — preserve it |

**The design job is to minimize extraneous load while preserving germane load.** You can't make complex information less complex — you can make it less confusing to look at.

---

## Practical Rules

### Chunk — ≤4 items per group

Miller's law: working memory holds 4±1 items comfortably. Seven is the theoretical limit; four is the comfortable limit for unfamiliar information.

- Navigation menus: group into clusters of ≤4, separated by dividers
- Feature lists: 3–4 per visible section, then continue below the fold
- Steps in a flow: ≤4 visible at once; progress indicator handles the rest
- Options in a dropdown: group if >7 items

**Bad:** 12-item flat navigation  
**Good:** 4 top-level items, sub-items revealed on hover/selection

### Spatial contiguity — label near its referent

Users should not have to scan between two separate areas to understand one thing. If a label and its data are on opposite sides of the screen, the user must hold one in memory while finding the other. That's extraneous load.

- Chart labels inside or adjacent to their data series — not in a separate legend across the chart
- Form hints below the field, not at the top of the form
- Tooltips appear next to the element they describe
- Error messages inline below the offending field

### Progressive disclosure — show minimum first

Show only what's needed for the current step. Reveal complexity on demand.

- Onboarding: one action per screen
- Settings: basic options visible; advanced options behind "Show more"
- Forms: single column, one field at a time for high-friction data (payment, medical)
- Dashboards: summary first, drilldown behind a click

**The worst version of a feature:** a form with 20 fields visible at once, even though most users only need 3.

### Consistent terminology

Calling the same thing by different names forces the user to wonder if they're different things. That's extraneous load.

- Pick one word for each concept — use it everywhere: header, button label, error message, documentation
- Don't alternate: "items" / "records" / "entries" — pick one
- Navigation labels match the page titles they link to

### Avoid split attention

When two pieces of information need to be understood together, present them together — not separated.

- Instruction text integrated into the form, not in a separate "How to use this" sidebar
- Code and its explanation in the same code block, not code above and description two paragraphs below
- Steps written in sequence, not "see steps below" with a jump

---

## 8-Item Checklist for Huashu Designs

Run before delivering any interactive prototype, onboarding flow, or information-dense slide deck.

- [ ] **No more than 4 concepts per screen/slide** — if there are more, split or layer
- [ ] **Labels are adjacent to what they describe** — no cross-screen label/data pairs
- [ ] **Navigation/controls are self-evident** — no instructions needed to use them
- [ ] **Primary action is visually dominant** — the most important button/CTA is unmissable
- [ ] **Multi-step flows show current position** — step N of N, progress bar, or breadcrumb
- [ ] **Only the current step's information is shown** — future steps don't distract
- [ ] **Consistent terminology throughout** — same concept, same word, everywhere
- [ ] **Empty states explain the next action** — "No projects yet — create one to get started" not just "No projects"

---

## Slide Deck Specific Rules

Slides have one cognitive advantage (fixed, focused attention) and one disadvantage (no interaction — the user can't slow down or re-read easily).

| Principle | Slide rule |
|---|---|
| Intrinsic load | **One core idea per slide.** If you can't say "this slide is about X" in one sentence, split it. |
| Extraneous load | Remove everything that doesn't directly support the core idea. No decoration for decoration's sake. |
| Chunking | Max 3 bullet points. If you have 5, make 2 slides. |
| Spatial contiguity | Annotations on the diagram, not in a separate legend. Speaker notes for supporting detail. |
| Progressive disclosure | Reveal complex diagrams in steps across multiple slides, not all at once. |

**The highest-cognitive-load slide:** a dense bullet list with 8 items and a 10-word header. The viewer's eye jumps, their working memory fills with items they can't contextualize, and they tune out.

---

## Onboarding Flow Rules

Onboarding is where cognitive load most often kills retention. New users have no schema for your product — everything is intrinsic load for them.

**Design for time-to-value, not feature coverage:**
- First screen: one action that shows the product's core value
- Don't front-load all configuration — defer until the user needs it
- Skip/Later for everything optional
- Scaffold context: show the user what they're building toward before asking them to do it

**5-step onboarding pattern:**
1. Welcome — one sentence on the product's core value proposition
2. One action — the simplest version of the primary use case
3. See a result — the user gets value before they finish setup
4. Optional configuration — personalization that improves the experience
5. Home state — land in a state with a clear next action (not a blank dashboard)

**Worst pattern:** account setup → preferences → team invite → integrations → done. Five steps of extraneous load before the user has seen the product work.

---

## When Load Is Appropriate

Not all cognitive engagement is a problem. **Germane load** — the processing that builds understanding — should be preserved.

- A slightly complex interaction that reveals the product's depth is appropriate for a power-user tool
- Onboarding that teaches a non-obvious concept through doing (not just reading) is appropriate
- Information density is appropriate on a data dashboard for analysts who want it

The rule is not "simpler is always better." The rule is: **eliminate load that comes from the design, preserve load that comes from the content.**
