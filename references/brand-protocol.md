# Core Asset Protocol (Brand Asset Protocol)

> **This is the #1 constraint in v1 and the backbone of output stability.** Whether the agent follows this protocol determines whether the output scores 40 or 90. Do not skip any step.
>
> **v1.1 Rewrite (2026-04-20):** Upgraded from "Brand Asset Protocol" to "Core Asset Protocol." The prior version over-focused on color values and fonts, missing the most fundamental design assets: logo, product photos, UI screenshots. Original rationale: "Beyond the so-called brand colors, we obviously need to find and use the DJI logo, use the Pocket 4 product photo. For non-physical products like websites or apps, the logo should at minimum be required. This is probably more fundamental than any brand design spec. Otherwise, what are we even expressing?"

**Load contract:** `SKILL.md` §1.a keeps only a compressed summary of this protocol. This file is the authoritative source for: (1) the 5-step procedure with per-asset-type search-path tables and three logo-download fallback paths; (2) the 5-10-2-8 quality gate and 8/10 scoring rubric; (3) the full `brand-spec.md` template; (4) the failure-case anecdotes; (5) the "30-min cost vs rework cost" rationale table. When SKILL.md directs you here, read this entire file before executing the protocol.

**Trigger:** The task involves a specific brand — the user mentioned a product name, company name, or explicit client (Stripe, Linear, Anthropic, Notion, Lovart, DJI, their own company, etc.), regardless of whether the user has proactively provided brand materials.

**Hard prerequisite:** Before starting the protocol, confirm via "#0 Facts Before Assumptions" that the brand/product exists and its status is known. If you're still uncertain whether the product has launched, what version it is, or its specs — go back and search first.

---

## Core Philosophy: Assets > Spec

**The essence of a brand is being recognized.** What drives recognition? Ranked by contribution:

| Asset type | Recognition contribution | Required? |
|---|---|---|
| **Logo** | Highest · any brand is instantly identified when the logo appears | **Required for every brand** |
| **Product photos / renders** | Very high · the "hero" of a physical product is the product itself | **Required for physical products (hardware / packaging / consumer goods)** |
| **UI screenshots / interface assets** | Very high · the "hero" of a digital product is its interface | **Required for digital products (App / website / SaaS)** |
| **Color values** | Medium · supporting recognition; weak on its own, often clashes with generic palettes | Supporting |
| **Typography** | Low · needs the above assets to build recognition | Supporting |
| **Personality keywords** | Low · for agent self-check only | Supporting |

**Translated into execution rules:**
- Extract only colors + fonts without finding logo / product photos / UI → **protocol violation**
- Use CSS silhouettes or hand-drawn SVG instead of real product images → **protocol violation** (the output is a "generic tech animation" — every brand looks the same)
- Can't find an asset, don't tell the user, don't AI-generate, just push through → **protocol violation**
- It is better to stop and ask the user for materials than to fill with generic placeholders

---

## 5-Step Hard Process (each step has a fallback; never skip silently)

### Step 1 · Ask (request the full asset checklist at once)

Don't just ask "Do you have brand guidelines?" — too broad; the user doesn't know what to provide. Ask by item:

```
For <brand/product>, which of the following do you have? Listed by priority:
1. Logo (SVG / high-res PNG) — required for any brand
2. Product photos / official renders — required for physical products (e.g. DJI Pocket 4 photos)
3. UI screenshots / interface assets — required for digital products (e.g. main app screens)
4. Color list (HEX / RGB / brand palette)
5. Font list (Display / Body)
6. Brand guidelines PDF / Figma design system / brand website URL

Share what you have; I'll search / scrape / generate the rest.
```

### Step 2 · Search Official Channels (by asset type)

| Asset | Search path |
|---|---|
| **Logo** | `<brand>.com/brand` · `<brand>.com/press` · `<brand>.com/press-kit` · `brand.<brand>.com` · inline SVG in the site header |
| **Product photos / renders** | `<brand>.com/<product>` product detail page hero + gallery · official YouTube launch film frames · official press release images |
| **UI screenshots** | App Store / Google Play product page screenshots · site screenshots section · official product demo video frames |
| **Color values** | Site inline CSS / Tailwind config / brand guidelines PDF |
| **Typography** | Site `<link rel="stylesheet">` references · Google Fonts trace · brand guidelines |

`WebSearch` fallback keywords:
- Logo not found → `<brand> logo download SVG`, `<brand> press kit`
- Product photo not found → `<brand> <product> official renders`, `<brand> <product> product photography`
- UI not found → `<brand> app screenshots`, `<brand> dashboard UI`

### Step 3 · Download Assets — 3 Fallback Paths Per Type

**3.1 Logo (required for every brand)**

Three paths, ordered by success rate:
1. Standalone SVG/PNG file (ideal):
   ```bash
   curl -o assets/<brand>-brand/logo.svg https://<brand>.com/logo.svg
   curl -o assets/<brand>-brand/logo-white.svg https://<brand>.com/logo-white.svg
   ```
2. Extract inline SVG from homepage HTML (needed in ~80% of cases):
   ```bash
   curl -A "Mozilla/5.0" -L https://<brand>.com -o assets/<brand>-brand/homepage.html
   # then grep <svg>...</svg> to extract the logo node
   ```
3. Official social media avatar (last resort): GitHub / Twitter / LinkedIn company avatars are typically 400×400 or 800×800 transparent PNG

**3.2 Product photos / renders (required for physical products)**

By priority:
1. **Official product page hero image** (highest priority): right-click to get URL / curl to download. Resolution is usually 2000px+
2. **Official press kit**: `<brand>.com/press` often has high-res product photo downloads
3. **Official launch video frames**: use `yt-dlp` to download YouTube video, then ffmpeg to extract high-res frames
4. **Wikimedia Commons**: public domain often has good options
5. **AI-generated fallback** (nano-banana-pro): use the real product photo as reference, ask AI to generate a variant suited to the animation context. **Do not use CSS/SVG hand-drawn replacements**

```bash
# Example: download DJI official product hero image
curl -A "Mozilla/5.0" -L "<hero-image-url>" -o assets/<brand>-brand/product-hero.png
```

**3.3 UI screenshots (required for digital products)**

- App Store / Google Play product screenshots (note: may be mockups rather than real UI — cross-check)
- Site screenshots section
- Product demo video frames
- Official product Twitter/X launch screenshots (often the most up-to-date)
- When the user has an account: directly screenshot the real product interface

**3.4 · The 5-10-2-8 Quality Gate (hard rule)**

> **Logo has a different rule from all other assets.** If you have a logo, you must use it (if you don't, stop and ask the user). All other assets (product photos / UI / reference images / supporting images) follow the 5-10-2-8 quality gate.
>
> Original rationale (2026-04-20): "Our principle is to search 5 rounds, find 10 assets, select 2 good ones. Each must score 8/10 or above. Better to have fewer than to pad the count to complete the task."

| Dimension | Standard | Anti-pattern |
|---|---|---|
| **5 rounds of search** | Multi-channel cross-search (official site / press kit / official social / YouTube frames / Wikimedia / user's own screenshot) — not stopping after 2 results on the first page | Using whatever shows up on the first search page |
| **10 candidates** | Collect at least 10 candidates before filtering | Grabbing 2 and having nothing to compare against |
| **Pick 2 good ones** | Select the best 2 from 10 as final assets | Using all 10 = visual overload + diluted quality signal |
| **Each must score 8/10+** | If it doesn't hit 8/10, **don't use it** — use an honest placeholder (gray block + text label) or AI-generated (nano-banana-pro using official reference as base) | Putting 7/10 filler into brand-spec.md |

**8/10 scoring dimensions** (record scores in `brand-spec.md`):

1. **Resolution** · ≥2000px (print / large screen: ≥3000px)
2. **Source clarity** · official source > public domain > free asset > suspected unauthorized copy (suspected copy = 0 immediately)
3. **Brand personality fit** · consistent with the "personality keywords" in brand-spec.md
4. **Light / composition / style consistency** · the 2 final assets should not clash when placed side-by-side
5. **Standalone narrative** · can carry a story role on its own (not just decoration)

**Why this gate is a hard rule:**
- The philosophy: **quality over quantity**. Filler assets are worse than nothing — they pollute visual taste and signal "unprofessional"
- **The "one detail at 120%, everything else at 80%" rule, quantified:** 8/10 is the floor for "everything else at 80%"; true hero assets need 9–10
- Every visual element either adds points or subtracts points from the viewer's impression. A 7/10 asset subtracts points — better to leave the space empty

**Logo exception (reiterated):** If you have it, you must use it. The 5-10-2-8 gate does not apply to the logo. The logo is not a "pick one from many" question — it is the foundation of brand recognition. Even a 6/10 logo is 10x better than no logo.

### Step 4 · Verify + Extract (not just grepping colors)

| Asset | Verification action |
|---|---|
| **Logo** | File exists + SVG/PNG opens + at least two versions (dark background / light background) + transparent background |
| **Product photos** | At least one at 2000px+ resolution + removed or clean background + multiple angles (main view, detail, scene) |
| **UI screenshots** | Genuine resolution (1x / 2x) + latest version (not outdated) + no user data visible |
| **Color values** | `grep -hoE '#[0-9A-Fa-f]{6}' assets/<brand>-brand/*.{svg,html,css} \| sort \| uniq -c \| sort -rn \| head -20`, filter out blacks/whites/grays |

**Watch for demo-brand contamination:** Product screenshots often include brand colors from the user's demo content (e.g., a design tool screenshot showing a client's red). That is not the tool's own color. **When two strong colors appear simultaneously, you must distinguish which is the product's.**

**Brand has multiple faces:** The same brand's marketing site colors and product UI colors often differ (e.g., Lovart's marketing site is warm beige + orange; the product UI is Charcoal + Lime). **Both are real** — choose the face that matches the deliverable context.

### Step 5 · Lock Down in `brand-spec.md`

The template is at `assets/brand-spec-template.md` — Read that file, copy its contents to create `<project-dir>/brand-spec.md`, and fill in the actual collected asset paths and color values.

**Execution discipline after writing the spec (hard requirements):**
- All HTML must **reference** asset file paths from `brand-spec.md` — no CSS silhouettes or hand-drawn SVG allowed
- Logo referenced as a real `<img>` file — not redrawn
- Product photos referenced as real `<img>` files — not replaced with CSS silhouettes
- CSS variables injected from spec: `:root { --brand-primary: ...; }` — HTML only uses `var(--brand-*)`
- This turns brand consistency from "relying on discipline" to "enforced by structure" — adding a color requires updating the spec first

---

## Global Fallback for Failures

Handle each asset type separately:

| Missing asset | Action |
|---|---|
| **Logo completely unfindable** | **Stop and ask the user** — do not push through (the logo is the foundation of brand recognition) |
| **Product photos (physical product) unfindable** | First try AI generation via nano-banana-pro (using official reference as base) → then ask the user to provide → last resort: honest placeholder (gray block + text label, clearly marked "product photo pending") |
| **UI screenshots (digital product) unfindable** | Ask the user to screenshot from their own account → official demo video frames. Do not use mockup generators to fill the gap |
| **Color values completely unfindable** | Use "design direction consultant mode" — present the user with 3 direction options, each clearly labeled as an assumption |

**Banned:** silently replacing missing assets with CSS silhouettes or generic gradients. That is the biggest anti-pattern in this protocol. **Better to stop and ask than to fill.**

---

## Failure Cases (real incidents)

- **Kimi animation:** Guessed from memory that "it should be orange" — Kimi is actually `#1783FF` blue. Required full redo.
- **Lovart design:** Mistook the red from a demo brand shown inside a product screenshot for Lovart's own brand color. Nearly ruined the entire design.
- **DJI Pocket 4 launch animation (2026-04-20, the incident that triggered this protocol upgrade):** Ran the old color-only protocol — didn't download the DJI logo, didn't find the Pocket 4 product photo, used CSS silhouettes instead. Output: "generic dark background + orange accent tech animation" with zero DJI recognition. Original quote: "Otherwise, what are we even expressing?" → protocol upgraded.
- Extracted colors but never wrote them into brand-spec.md — by slide 3, forgot the primary hex, added a "close but not quite" hex on the fly — brand consistency collapsed.

## Protocol Cost vs. Cost of Skipping

| Scenario | Time |
|---|---|
| Correctly completing the protocol | 5 min to download logo + 10 min for 3–5 product/UI assets + 5 min to grep colors + 10 min to write spec = **30 minutes** |
| Cost of skipping the protocol | Generic unrecognizable animation → 1–2 hours of user rework, potentially a full redo |

**This is the cheapest investment in stability.** Especially for paid work, launch events, or important client projects — 30 minutes on the asset protocol is insurance.
