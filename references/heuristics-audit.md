# Heuristics Audit: Nielsen's 10 + Scoring Framework

> Use when running post-delivery critique on interactive prototypes, app mockups, or any design where a user performs tasks. Complements `references/critique-guide.md` (philosophy/hierarchy/craft) — this file adds the **usability** layer.

**When to load:** user requests critique; you're reviewing an app prototype, interactive demo, or onboarding flow; the design involves user input, navigation, or multi-step flows.

---

## Scoring Scale

| Score | Meaning |
|---|---|
| **4** | Exemplary — actively helps users |
| **3** | Good — no violation |
| **2** | Minor violation — friction, but recoverable |
| **1** | Major violation — users will fail or give up |
| **0** | Not applicable to this design |

---

## The 10 Heuristics

### H1 · Visibility of System Status

Keep users informed about what is happening, with feedback in a reasonable time.

**Checklist:**
- Loading states shown for any action >300ms
- Active/selected state is visually distinct from default
- Progress shown for multi-step flows (step N of N)
- After a form submit: success or error confirmed before moving on
- File uploads and async operations have progress indication

**Slide/demo note:** cinematic animations don't need H1. App prototypes and interactive dashboards do — a "send" button with no response is a H1 failure.

---

### H2 · Match Between System and Real World

Use language and concepts familiar to the user — not system terminology.

**Checklist:**
- Labels use user vocabulary, not internal code names or database field names
- Icons map to familiar real-world metaphors (folder = collection, trash = delete)
- Dates/numbers formatted for the user's locale
- Error messages describe what happened in plain language (not "Error 403")
- Actions are described by their outcome, not their mechanism ("Save draft" not "POST /drafts")

---

### H3 · User Control and Freedom

Users make mistakes. Provide clearly marked emergency exits and undo paths.

**Checklist:**
- Destructive actions are reversible, or have a confirmation step
- "Back" always works and returns to the previous meaningful state
- Cancel is available during any multi-step flow
- Users can dismiss modals/toasts without completing the action
- No action locks the user into a state with no exit

**Red flag:** a dialog with only a "Confirm" button and no cancel.

---

### H4 · Consistency and Standards

Follow platform conventions. Don't make users learn your vocabulary when a standard exists.

**Checklist:**
- Interactive elements look interactive (links underlined or colored, buttons elevated)
- Non-interactive elements don't look interactive
- Terminology is consistent throughout (same thing = same name)
- Navigation pattern matches platform convention (iOS: tab bar + nav stack; web: top nav or sidebar)
- Form validation behavior is consistent (all fields validate at the same point in the flow)

---

### H5 · Error Prevention

Design to prevent problems from occurring, rather than just handling them after.

**Checklist:**
- Inputs that accept specific formats show format hints inline (not only on error)
- Date pickers prevent invalid date entry
- Destructive actions require confirmation ("Delete 12 items?" not just "Delete")
- Required fields are marked before submission
- Password fields show character count or strength meter

**Better than error recovery:** removing the possibility of the error.

---

### H6 · Recognition Rather Than Recall

Minimize memory load. Users shouldn't have to remember information from one screen to use another.

**Checklist:**
- Previously entered information is visible during multi-step flows
- Options are visible in context (dropdown) rather than requiring free recall (text input)
- Breadcrumbs or persistent context show where the user is in a flow
- "Recently used" or defaults surface the most likely choices
- Tooltips/labels appear on hover for icon-only controls

---

### H7 · Flexibility and Efficiency of Use

Serve both novice and expert users. Accelerators for experts shouldn't block beginners.

**Checklist:**
- Keyboard shortcuts available for frequent actions (and discoverable via tooltip)
- Expert paths exist (bulk actions, command palette, keyboard nav)
- Defaults serve the most common case — advanced options don't clutter the primary flow
- Search as an alternative to navigation for large datasets

**Prototype context:** for interactive demos, at minimum ensure the primary (beginner) path is obvious. Expert shortcuts are a bonus.

---

### H8 · Aesthetic and Minimalist Design

Every extra element competes with the relevant information. Omit the irrelevant.

**Checklist:**
- No decorative text, icon, or element that carries no functional or informational value
- Each screen has a single primary action — secondary actions are subordinate
- Help text appears only when needed (on hover, on focus, on error) — not permanently
- Modals contain only what's needed to complete one task

**Connection to anti-slop rule:** H8 is the heuristic version of the anti-slop principle. "Would the design get worse if I removed this?" — if no, remove it.

---

### H9 · Help Users Recognize, Diagnose, and Recover From Errors

Error messages must tell the user what happened, why, and what to do next.

**Error message formula:** `[What happened] + [Why] + [How to fix]`

| ❌ Bad | ✅ Good |
|---|---|
| "An error occurred" | "Couldn't save — your session expired. Sign in again to continue." |
| "Invalid input" | "Password must be at least 8 characters. Yours has 5." |
| "403 Forbidden" | "You don't have permission to edit this item. Ask an admin to grant access." |
| "File error" | "This file is 24 MB. Maximum is 10 MB. Compress the file or upload a smaller one." |

**Visual treatment:**
- Error text in red, inline below the offending field (not only in a toast)
- Error state persists until resolved — don't auto-dismiss field errors
- Preserve user's input — don't clear a form on validation error

---

### H10 · Help and Documentation

When help is needed, make it easy to find and focused on the user's task.

**Checklist:**
- Empty states explain what to do next (not just "No items found")
- Onboarding for non-obvious workflows (not assumed knowledge)
- Complex inputs have inline examples or hints ("e.g. jonas@company.com")
- Help links open without losing user's current state (side panel or new tab)

**Prototype scope:** H10 is a bonus in demos — include empty-state copy and one inline hint per complex field.

---

## Audit Output Template

```
## Heuristics Audit

| Heuristic | Score | Issue (if score < 3) |
|---|---|---|
| H1 · System Status | X | |
| H2 · Real World Match | X | |
| H3 · User Control | X | |
| H4 · Consistency | X | |
| H5 · Error Prevention | X | |
| H6 · Recognition | X | |
| H7 · Flexibility | X | |
| H8 · Minimal Design | X | |
| H9 · Error Recovery | X | |
| H10 · Help | X | |

**Heuristic score**: X.X/10 (average of applicable heuristics)

**Critical violations (score 1):**
[List each: heuristic, specific location in design, concrete fix]

**Minor friction (score 2):**
[List each: heuristic, specific location, suggested fix]
```

---

## When to Combine With critique-guide.md

Run both when critiquing app prototypes or interactive demos:
- `critique-guide.md` → philosophy coherence, visual hierarchy, craft, originality
- `heuristics-audit.md` → usability: can a user actually accomplish their goal?

For static slides, infographics, or cinematic animations: `critique-guide.md` only. H1–H10 don't apply to non-interactive output.
