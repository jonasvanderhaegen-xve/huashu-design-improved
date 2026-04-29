# Edge-Case Hardening

> Run before delivering any interactive prototype or HTML demo that will be handed off, demoed to stakeholders, or used as a production reference. Murphy's law: the one data case you didn't test is the one that appears during the demo.

---

## Text Length Edge Cases

The most common source of broken layouts in prototypes. Design with perfect-length placeholder text, then test with extremes.

### Long text

| Element | Long-text failure mode | Fix |
|---|---|---|
| Card title | Overflows card, breaks layout | `overflow: hidden; text-overflow: ellipsis; white-space: nowrap;` or `display: -webkit-box; -webkit-line-clamp: 2;` |
| Button label | Button grows wider than its container | `max-width`; truncate with ellipsis or wrap to 2 lines |
| Table cell | Breaks column alignment | Fixed column widths + overflow truncation |
| Nav item | Pushes other items off screen | `overflow: hidden` on nav container; test with translated strings (German is ~30% longer than English) |
| Badge/chip | Grows unbounded | Max-width + truncate |
| Tooltip | Overflows viewport edge | Flip direction: bottom → top, right → left |

```css
/* Single-line truncation */
.truncate-1 {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Multi-line truncation (2 lines) */
.truncate-2 {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}
```

### Short/missing text

| Scenario | What breaks | Fix |
|---|---|---|
| Empty user name | Layout collapses, avatar label is blank | Fallback: first letter of email, or "?" |
| 1-character value | Looks wrong in a cell sized for 10 chars | Test with "A", "1", "–" |
| Empty list | No items → blank space, no explanation | Always write an empty state (see `content-guidelines.md` §UX Writing) |
| Zero count in metric | "0" in a card designed for "1,234" | Ensure zero doesn't look like a loading failure |

### Special characters

Test your inputs and displays with:
- Apostrophes and quotes: `O'Brien`, `"quoted"`, `it's`
- Ampersands and angle brackets: `AT&T`, `<script>`, `a > b`
- Accented characters: `José`, `Ångström`, `Müller`
- Emoji in user content: `Jonas 🎉`, `🚀 Launch`
- Long unbreakable strings: `averylongwordwithnospacesthatbreakslayout`
- Currency and numbers: `€1.234,56`, `¥100,000`, `$0.001`

---

## Data Volume Edge Cases

### Zero items

Every list, table, or grid needs an explicit empty state design.

```jsx
{items.length === 0 ? (
  <EmptyState
    title="No projects yet"
    description="Create your first project to get started"
    action={<Button>New project</Button>}
  />
) : (
  <ItemList items={items} />
)}
```

### One item

Layouts designed for grids often look wrong with a single item. A 3-column grid with one card usually looks abandoned. Test and decide: keep the grid (with the lone card left-aligned) or switch to a single-column layout below a threshold.

### Maximum items

| Element | Maximum to test | Common failure |
|---|---|---|
| Dropdown | 100+ options | No scroll, or scroll locks viewport |
| Table | 500+ rows | Renders all rows, freezes browser |
| Tag/chip list | 20+ tags | Wraps unpredictably, overflows container |
| Navigation | 10+ items | Runs off screen on mobile |
| Select/combobox | 1000+ options | Use virtualization |

For demos: limit data to a believable amount but ensure the component handles overflow gracefully (scroll, pagination, virtual list).

### Simultaneous operations

| Scenario | What breaks |
|---|---|
| Double-click "Save" | Two simultaneous saves — second may overwrite or duplicate |
| Click "Delete" then immediately "Undo" | Race condition — item may be deleted before undo registers |
| Navigate away during file upload | Upload cancels silently, no feedback |
| Rapid tab switching | Stale data rendered for wrong tab |

For prototypes: add `pointer-events: none` on the triggering element during async operations. Show a loading state. Re-enable on completion.

---

## Error Scenarios

Every async action can fail. Design the failure state, not just the success state.

| Action | Failure states to design |
|---|---|
| Form submit | Validation error (field-level) + server error (form-level) |
| File upload | Too large / wrong format / network failure during upload |
| Data fetch | Network offline / server 500 / empty result |
| Authentication | Wrong credentials / account locked / session expired |
| Payment | Card declined / expired / network timeout |
| Delete/destructive action | Dependency error ("Can't delete — 3 active campaigns use this") |

**Minimum for a huashu prototype:**
- One inline validation error (field-level, per H9 in `heuristics-audit.md`)
- One full-screen or modal error for a failed async action
- One empty state for a successful fetch that returned no results

---

## i18n Hardening

For prototypes that will be used in multiple languages or regions:

### Text expansion

Translations are longer than English. Reserve 30–40% extra space for European languages, 100%+ for some languages.

| Language | Expansion vs English |
|---|---|
| German | +30–35% |
| French | +15–20% |
| Spanish | +15–25% |
| Russian | +15–20% |
| Arabic | −10% (but RTL!) |
| Chinese/Japanese | −20–30% (more compact) |

**Test:** temporarily set all UI strings to `"MMMMMMMMMMMM"` — if nothing breaks at maximum width, your layout is robust.

### RTL layouts (Arabic, Hebrew, Persian)

```css
/* Logical properties — flip automatically in RTL */
.card {
  padding-inline-start: 16px;   /* left in LTR, right in RTL */
  padding-inline-end: 24px;
  border-inline-start: 3px solid var(--accent);
  margin-inline-end: 8px;
}

/* Flip icons that imply direction */
[dir="rtl"] .arrow-icon,
[dir="rtl"] .chevron-icon {
  transform: scaleX(-1);
}
```

### Number, date, and currency formatting

Use the browser's `Intl` API — never hardcode formats.

```javascript
// Dates
new Intl.DateTimeFormat('de-DE').format(date)  // → "29.4.2026"
new Intl.DateTimeFormat('en-US').format(date)  // → "4/29/2026"

// Numbers
new Intl.NumberFormat('de-DE').format(1234567)  // → "1.234.567"
new Intl.NumberFormat('en-US').format(1234567)  // → "1,234,567"

// Currency
new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(1234.56)
// → "1.234,56 €"
```

**Prototype note:** use `Intl` even in demos — hardcoded `$1,234` looks wrong in a prototype pitched to a European client.

---

## Pre-Delivery Hardening Checklist

Run through these before handing off any interactive prototype:

**Text edge cases:**
- [ ] Long title doesn't break card/component layout
- [ ] Empty name/label has a fallback
- [ ] Special characters (apostrophe, ampersand, emoji) render correctly

**Data volume:**
- [ ] Empty list shows an empty state (not blank)
- [ ] 1-item list looks intentional
- [ ] Large list is scrollable (not frozen)

**Error states:**
- [ ] At least one form field shows a validation error
- [ ] At least one async failure shows an error message (not silent)
- [ ] Destructive action has a confirmation with the object name

**Interactive:**
- [ ] Double-click / rapid repeat action is handled (loading state prevents)
- [ ] Disabled state on form submit during loading
- [ ] Back button returns to the correct state

**Layout:**
- [ ] Longest expected string doesn't break layout
- [ ] Smallest expected value (0, 1 item, empty) looks right
- [ ] Fixed/sticky elements don't cover content on scroll
