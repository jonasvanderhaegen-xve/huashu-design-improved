# Design Personas

> Use as a critique lens when reviewing interactive prototypes and app mockups. Personas surface failure modes that are invisible when you design only for yourself or for the "ideal" user. Not a research deliverable — a practical checklist for stress-testing designs before delivery.

---

## When to Use

- Post-delivery critique of interactive prototypes (run alongside `heuristics-audit.md`)
- When a design must work for a broad or unknown audience
- When the client's users are not the same as the designer or reviewer
- When the design involves forms, onboarding, or multi-step flows

Not needed for: cinematic animations, brand films, static slide decks. Use for anything where a real user performs a task.

---

## The 5 Archetypes

### P1 · The Impatient Expert

**Profile:** Has used many products like this before. Knows what they want. Will not read instructions. Will abandon if the path to their goal isn't obvious in 5 seconds.

**Key behaviors:**
- Skips onboarding entirely
- Uses keyboard shortcuts before checking if they exist
- Expects the primary action to be immediately accessible, not buried in a flow
- Resents confirmation dialogs for actions they do frequently

**Questions to ask the design:**
- Can an expert reach the primary action in ≤2 clicks from any screen?
- Do keyboard shortcuts exist for repeat actions? (H7 — flexibility)
- Are confirmation dialogs avoidable for non-destructive actions (e.g. skip for undo-able deletes)?
- Is there a search or command palette for power navigation?

**Common failure:** a 5-step onboarding that cannot be skipped. The expert abandons at step 2.

---

### P2 · The Confused First-Timer

**Profile:** Has never used a product like this. May not understand domain terminology. Will read instructions but lose patience if there are too many. Makes wrong assumptions about where things are.

**Key behaviors:**
- Reads button labels carefully — unexpected verbs cause hesitation
- Interprets icons literally, not metaphorically
- Will look for a tutorial or "how does this work?" link
- Gets stuck at any ambiguous fork in a flow

**Questions to ask the design:**
- Are button labels verbs + objects ("Create project") not nouns alone ("Project")?
- Do icons have visible labels, at least on first encounter?
- Is the first-time empty state instructive ("Create your first X") not just empty?
- Is there a visible path to help when stuck? (H10 — help)
- Does the onboarding show value before asking for configuration?

**Common failure:** icon-only navigation on first use. The expert knows the trash icon means delete. The first-timer doesn't know if it means "trash" (view trash) or "delete" (permanently remove).

---

### P3 · The Mobile-First User

**Profile:** Primarily or exclusively on a phone. May be on a slow connection. Has limited screen space and uses thumbs. Switching between apps mid-task is normal.

**Key behaviors:**
- Taps, doesn't hover
- Loses context if a flow requires too many steps (gets interrupted, forgets where they were)
- Abandons if text input is required for more than one field without a clear payoff
- Expects content to load fast; gives up after ~3 seconds

**Questions to ask the design:**
- Are all tap targets ≥ 44px? (see `responsive-design.md`)
- Does the flow survive interruption at any step? (resumable state)
- Are text inputs minimized — can any field use a picker, toggle, or default instead?
- Does the primary action work thumb-reachably (bottom 2/3 of screen)?
- Does content load progressively, or does the user stare at blank until everything is ready?

**Common failure:** a form that requires 8 typed inputs on mobile with no autofill support. The user fills 5, gets a phone call, comes back, and the form is blank.

---

### P4 · The Cautious User

**Profile:** Moves slowly and deliberately. Reads before clicking. Afraid of making a mistake, especially with irreversible actions. Does not trust that they can undo.

**Key behaviors:**
- Will not click a button whose effect is ambiguous
- Reads destructive action confirmations fully
- Looks for "are you sure?" dialogs before proceeding
- Will stop and ask for help rather than risk a mistake

**Questions to ask the design:**
- Are destructive actions clearly labeled with what will be destroyed ("Delete 5 projects" not "Delete")?
- Is undo available for all reversible actions?
- Are irreversible actions flagged as such ("This cannot be undone")?
- Are there visible escape routes at every step? (H3 — user control)
- Does the confirmation dialog explain consequences, not just confirm the action?

**Common failure:** a "Delete account" option in settings with the same visual weight as "Edit profile." The cautious user is scared to touch settings at all.

---

### P5 · The Low-Bandwidth / Low-Spec User

**Profile:** On a slow connection, an older device, or a constrained environment (public WiFi, corporate proxy, accessibility tool). Cannot rely on fast loads or smooth animations.

**Key behaviors:**
- Waits longer for loads but expects feedback that something is happening
- May have browser extensions that alter the page (ad blockers, screen readers)
- May have motion sensitivity settings enabled
- May use the keyboard instead of the mouse (motor impairment, preference)

**Questions to ask the design:**
- Is there loading feedback for any action that takes >300ms?
- Do animations respect `prefers-reduced-motion`?
- Can the full flow be completed by keyboard alone? (Tab order, Enter to submit, Escape to cancel)
- Does the design work without custom fonts loaded? (fallback font should maintain layout)
- Are images lazy-loaded? Does the layout shift when they load?

```css
/* Respect motion preferences */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Common failure:** a purely mouse-driven flow. Tab → nothing happens. A keyboard user is immediately blocked.

---

## Persona Critique Template

Add this section to a critique when the design is an interactive prototype:

```
## Persona Stress-Test

| Persona | Critical failure? | Issue |
|---|---|---|
| P1 · Impatient Expert | ✅ / ⚠️ / ❌ | |
| P2 · Confused First-Timer | ✅ / ⚠️ / ❌ | |
| P3 · Mobile-First | ✅ / ⚠️ / ❌ | |
| P4 · Cautious | ✅ / ⚠️ / ❌ | |
| P5 · Low-Bandwidth | ✅ / ⚠️ / ❌ | |

✅ No blockers  ⚠️ Friction but completable  ❌ Cannot complete primary task
```

For huashu demos: P1 and P2 are the highest-priority archetypes — most stakeholder demos involve someone who either knows the product space well or is seeing it for the first time. P3 applies for any mobile prototype. P4 and P5 are worth a quick scan but not mandatory on every deliverable.

---

## Using Personas in Design Decisions

Personas are most useful when they create a conflict you have to resolve:

> "This modal is efficient for P1 but blocks P2 from understanding what just happened."

That tension is the design problem. Solve it (contextual help on hover, progressive disclosure, inline success message) rather than optimizing for one persona and ignoring the other.

Not every design needs to work equally well for all five. A developer tool optimizes for P1 and can safely deprioritize P4. A medical intake form optimizes for P4 and P2, and can accept that P1 finds it slow. **Explicitly deciding which personas matter for this product is more useful than trying to satisfy all five equally.**
