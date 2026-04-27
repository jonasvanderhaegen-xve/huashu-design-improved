# Design Context: Starting from Existing Context

**This is the most important one thing about this skill.**

Good hi-fi design always grows from existing design context. **Designing hi-fi from scratch is a last resort — it will always produce generic output.** So at the start of every design task, first ask: is there anything I can reference?

## What Is Design Context

In descending priority order:

### 1. User's Design System / UI Kit
The component library, color tokens, typeface specs, and icon system already in the user's product. **The ideal scenario.**

### 2. User's Codebase
If the user shares a codebase, it contains living component implementations. Read those files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Specific components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Read the code and copy exact values:** hex codes, spacing scale, font stack, border radius. Don't redraw from memory.

### 3. User's Deployed Product
If the user has a live product but no code, use Playwright or ask the user for screenshots.

```bash
# take a Playwright screenshot of a public URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

This shows you the real visual vocabulary.

### 4. Brand Guide / Logo / Existing Assets
The user may have: logo files, brand color specs, marketing materials, slide templates. All of this is context.

### 5. Competitor References
If the user says "like that XX website" — ask them to provide the URL or a screenshot. **Don't** work from a vague memory in your training data.

### 6. A Known Design System (fallback)
If none of the above exist, use a recognized design system as a base:
- Apple HIG
- Material Design 3
- Radix Colors (color)
- shadcn/ui (components)
- Tailwind default palette

Tell the user explicitly what you're using — let them know it's a starting point, not the final.

## The Process for Gathering Context

### Step 1: Ask the User

Mandatory questions at task start (from `workflow.md`):

```markdown
1. Do you have an existing design system / UI kit / component library? Where is it?
2. Do you have a brand guide, color / typeface specs?
3. Can you give me a screenshot or URL of your existing product?
4. Is there a codebase I can read?
```

### Step 2: When the User Says "No", Help Them Look

Don't give up immediately. Try:

```markdown
Let me see if there are any clues:
- Do any of your previous projects have related designs?
- What colors / typefaces does your company's marketing site use?
- What style is your product's logo? Can you give me a copy?
- Is there a product you admire that could serve as a reference?
```

### Step 3: Read All Context You Can Find

If the user provides a codebase path, read:
1. **List the file structure first:** find style/theme/component-related files
2. **Read theme/token files:** lift specific hex/px values
3. **Read 2–3 representative components:** see the visual vocabulary (hover state, shadow, border, padding pattern)
4. **Read global stylesheet:** base resets, font loading
5. **If there's a Figma link / screenshot:** look at it, but **trust the code more**

**Important:** **Don't** glance at it and work from impression. You've truly lifted the context only when you have 30+ specific values.

### Step 4: Vocalize the System You'll Use

After reviewing the context, tell the user the system you'll use:

```markdown
Based on your codebase and product screenshots, here's the design system I've extracted:

**Color**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typefaces**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all 8px border-radius
- Card: white background, subtle shadow, no border

I'll work from this system. Confirm this looks right?
```

Wait for user confirmation before starting.

## Designing From Scratch (fallback when there's no context)

**Strong warning:** Output quality will drop significantly in this case. Tell the user explicitly.

```markdown
You don't have design context, so I can only work from generic intuition.
Output will be "looks OK but lacks distinctiveness".
Would you like to continue, or should we gather some reference materials first?
```

If the user insists, make decisions in this order:

### 1. Pick an aesthetic direction
Don't produce a generic result. Pick a specific direction:
- brutally minimal
- editorial/magazine
- brutalist/raw
- organic/natural
- luxury/refined
- playful/toy
- retro-futuristic
- soft/pastel

Tell the user which one you chose.

### 2. Pick a known design system as the skeleton
- Use Radix Colors for the palette (https://www.radix-ui.com/colors)
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Use Tailwind spacing scale (multiples of 4)

### 3. Pick a distinctive font pairing

Don't use Inter/Roboto. Suggested pairings (all free on Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note: Fraunces is already overused by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every key decision has a rationale

Don't choose silently. Document in the HTML comment:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (when user provides a codebase)

If the user says "import this codebase as reference":

### Small (<50 files)
Read everything and internalize the context.

### Medium (50–500 files)
Focus on:
- `src/components/` or `components/`
- All styles/tokens/theme-related files
- 2–3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Ask the user to specify focus:
- "I'm building the settings page" → read existing settings-related files
- "I'm adding a new feature" → read the overall shell + closest reference
- Don't try to be comprehensive — be targeted

## Working with Figma / Design Files

If the user provides a Figma link:

- **Don't** expect to "convert Figma to HTML" directly — that requires additional tooling
- Figma links are usually not publicly accessible
- Ask the user to: export as **screenshots** and send to you + tell you specific color/spacing values

If given only a Figma screenshot, tell the user:
- I can see the visual, but can't extract precise values
- For key numbers (hex, px) please tell me directly, or export as code (Figma supports this)

## Final Reminder

**The quality ceiling of any project's design is determined by the quality of context you receive.**

10 minutes collecting context is more valuable than 1 hour designing hi-fi from scratch.

**When there's no context, ask the user for it first — don't just power through.**
