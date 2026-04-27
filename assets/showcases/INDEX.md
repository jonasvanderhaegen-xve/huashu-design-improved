# Design Philosophy Showcases — Sample Asset Index

> 8 scenes × 3 styles = 24 pre-built design samples
> Used in Phase 3 when recommending a design direction, to show directly "what this style looks like in practice"

## Style Reference

| Code | School | Style Name | Visual Character |
|------|--------|-----------|-----------------|
| **Pentagram** | Information Architecture | Pentagram / Michael Bierut | B&W restraint, Swiss grid, strong type hierarchy, #E63946 red accent |
| **Build** | Minimalism | Build Studio | luxury-grade whitespace (70%+), subtle weight (200–600), #D4A574 warm gold, refined |
| **Takram** | Eastern Philosophy | Takram | soft tech feel, natural colors (beige/grey/green), rounded corners, charts as art |

## Scene Quick Reference

### Content Design Scenes

| # | Scene | Specs | Pentagram | Build | Takram |
|---|-------|-------|-----------|-------|--------|
| 1 | WeChat Article Cover | 1200×510 | `cover/cover-pentagram` | `cover/cover-build` | `cover/cover-takram` |
| 2 | PPT Data Page | 1920×1080 | `ppt/ppt-pentagram` | `ppt/ppt-build` | `ppt/ppt-takram` |
| 3 | Vertical Infographic | 1080×1920 | `infographic/infographic-pentagram` | `infographic/infographic-build` | `infographic/infographic-takram` |

### Website Design Scenes

| # | Scene | Specs | Pentagram | Build | Takram |
|---|-------|-------|-----------|-------|--------|
| 4 | Personal Homepage | 1440×900 | `website-homepage/homepage-pentagram` | `website-homepage/homepage-build` | `website-homepage/homepage-takram` |
| 5 | AI Directory Site | 1440×900 | `website-ai-nav/ainav-pentagram` | `website-ai-nav/ainav-build` | `website-ai-nav/ainav-takram` |
| 6 | AI Writing Tool | 1440×900 | `website-ai-writing/aiwriting-pentagram` | `website-ai-writing/aiwriting-build` | `website-ai-writing/aiwriting-takram` |
| 7 | SaaS Landing Page | 1440×900 | `website-saas/saas-pentagram` | `website-saas/saas-build` | `website-saas/saas-takram` |
| 8 | Developer Docs | 1440×900 | `website-devdocs/devdocs-pentagram` | `website-devdocs/devdocs-build` | `website-devdocs/devdocs-takram` |

> Each entry includes both a `.html` (source) and `.png` (screenshot) file

## Usage Guide

### Reference During Phase 3 Recommendation
After recommending a design direction, show the pre-built screenshot for the matching scene:
```
"Here's the Pentagram style applied to a WeChat Article Cover → [show cover/cover-pentagram.png]"
"This is how Takram style looks on a PPT data page → [show ppt/ppt-takram.png]"
```

### Scene Matching Priority
1. User's requested scene has an exact match → show the corresponding scene directly
2. No exact match but similar type → show the closest scene (e.g. "product website" → show SaaS landing page)
3. No match at all → skip pre-built samples, proceed directly to Phase 3.5 live generation

### Side-by-Side Comparison Display
All 3 styles for the same scene work well displayed side by side, helping users compare intuitively:
- "This is the same WeChat Article Cover implemented in 3 different styles"
- Display order: Pentagram (rational restraint) → Build (luxury minimalism) → Takram (soft warmth)

## Content Details

### WeChat Article Cover (`cover/`)
- Content: Claude Code Agent workflow — 8-parallel Agent architecture
- Pentagram: large red "8" + Swiss grid lines + data bars
- Build: ultra-thin weight "Agent" floating in 70% whitespace + warm gold hairline
- Takram: 8-node radial flowchart as artwork + beige base

### PPT Data Page (`ppt/`)
- Content: GLM-4.7 open-source model coding breakthrough (AIME 95.7 / SWE-bench 73.8% / τ²-Bench 87.4)
- Pentagram: 260px "95.7" anchor + red/grey/light-grey contrast bar chart
- Build: three sets of 120px ultra-thin numbers floating + warm gold gradient contrast bars
- Takram: SVG radar chart + three-color overlay + rounded data cards

### Vertical Infographic (`infographic/`)
- Content: AI memory system CLAUDE.md optimized from 93KB to 22KB
- Pentagram: large "93→22" numbers + numbered blocks + CSS data bars
- Build: extreme whitespace + soft shadow cards + warm gold connecting lines
- Takram: SVG ring chart + organic curve flowchart + frosted glass cards

### Personal Homepage (`website-homepage/`)
- Content: portfolio homepage for indie developer Alex Chen
- Pentagram: 112px large name + Swiss grid columns + editorial numerals
- Build: glass-morphism nav + floating stat cards + ultra-thin weight
- Takram: paper texture + small circular avatar + hairline dividers + asymmetric layout

### AI Directory Site (`website-ai-nav/`)
- Content: AI Compass — 500+ AI tools directory
- Pentagram: square-corner search box + numbered tool list + uppercase category labels
- Build: rounded search box + refined white tool cards + pill labels
- Takram: organic offset card layout + soft category labels + chart-style connections

### AI Writing Tool (`website-ai-writing/`)
- Content: Inkwell — AI writing assistant
- Pentagram: 86px large headline + wireframe editor model + grid feature columns
- Build: floating editor card + warm gold CTA + luxury writing experience
- Takram: poetic serif headline + organic editor + flowchart

### SaaS Landing Page (`website-saas/`)
- Content: Meridian — business intelligence analytics platform
- Pentagram: black/white split columns + structured dashboard + 140px "3x" anchor
- Build: floating dashboard cards + SVG area chart + warm gold gradient
- Takram: rounded bar chart + flow nodes + soft earth tones

### Developer Docs (`website-devdocs/`)
- Content: Nexus API — unified AI model gateway
- Pentagram: left sidebar nav + square-corner code blocks + red string highlighting
- Build: centered floating code card + soft shadow + warm gold icons
- Takram: beige code blocks + flowchart connections + dashed feature cards

## File Stats

- HTML source files: 24
- PNG screenshots: 24
- Total assets: 48 files

---

**Version**: v1.0
**Created**: 2026-02-13
**For**: design-philosophy skill Phase 3 recommendation phase
