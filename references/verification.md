# Verification: Output Verification Workflow

Some design-agent native environments (e.g. Claude.ai Artifacts) have a built-in `fork_verifier_agent` that spins up a subagent and uses iframe screenshots to inspect output. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this built-in capability — doing it manually with Playwright covers the same verification scenarios.

## Verification Checklist

Run through this checklist after every HTML output:

### 1. Browser Render Check (required)

The most basic question: **can the HTML be opened?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use a Playwright screenshot (next section).

### 2. Console Error Check

The most common issue in HTML files is a JS error causing a white screen. Run a Playwright pass:

```bash
python ~/.claude/skills/huashu-design/scripts/verify.py path/to/design.html
```

This script will:
1. Open the HTML in headless Chromium
2. Save a screenshot to the project directory
3. Capture console errors
4. Report status

See `scripts/verify.py` for details.

### 3. Multi-Viewport Check

For responsive designs, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, button toggles — static screenshots won't catch these. **Recommend having the user open the browser and click through it themselves**, or use Playwright screen recording:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-Slide Check

For deck-type HTML, screenshot each slide:

```bash
python verify.py deck.html --slides 10  # screenshot first 10 slides
```

Generates `deck-slide-01.png`, `deck-slide-02.png`... for easy review.

## Playwright Setup

First-time setup:

```bash
# if not yet installed
npm install -g playwright
npx playwright install chromium

# or Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, use it directly.

## Screenshot Best Practices

### Full-page screenshot

```python
page.screenshot(path='full.png', full_page=True)
```

### Viewport screenshot

```python
page.screenshot(path='viewport.png')  # captures only the visible area by default
```

### Specific element screenshot

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-DPI screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for animation to settle

```python
page.wait_for_timeout(2000)  # wait 2s for animation to settle
page.screenshot(...)
```

## Sharing Screenshots with Users

### Open local screenshot directly

```bash
open screenshot.png
```

The user sees it in their own Preview / Figma / VSCode / browser.

### Upload to image host and share link

For sharing with remote collaborators (e.g. Slack / Lark / WeChat), have the user use their own image hosting tool or an MCP upload:

```bash
python ~/Documents/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link that can be pasted anywhere.

## When Verification Fails

### Page white screen

There will be an error in the console. Check first:

1. Whether the React+Babel script tag integrity hash is correct (see `react-setup.md`)
2. Whether there's a `const styles = {...}` naming conflict
3. Whether cross-file components are exported to `window`
4. JSX syntax errors (babel.min.js may not surface them — swap to unminified babel.js for clearer error messages)

### Animation laggy

- Record a segment in Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflow)
- Prefer `transform` and `opacity` for animation (GPU-accelerated)

### Font wrong

- Check whether `@font-face` URLs are accessible
- Check fallback fonts
- CJK fonts load slowly: show fallback first, switch when loaded

### Layout misaligned

- Check whether `box-sizing: border-box` is applied globally
- Check `* { margin: 0; padding: 0 }` reset
- Open gridlines in Chrome DevTools to inspect actual layout

## Verification = The Designer's Second Pair of Eyes

**Always do a pass yourself.** When AI writes code, it commonly produces:

- Looks correct but interaction has a bug
- Static screenshot fine but misaligned on scroll
- Looks great at wide viewport but breaks narrow
- Dark mode never tested
- Some components don't respond after Tweaks are switched

**One final minute of verification can save one hour of rework.**

## Common Verification Script Commands

```bash
# basic: open + screenshot + capture errors
python verify.py design.html

# multiple viewports
python verify.py design.html --viewports 1920x1080,375x667

# multiple slides
python verify.py deck.html --slides 10

# output to a specific directory
python verify.py design.html --output ./screenshots/

# headless=false, open a real browser for you to inspect
python verify.py design.html --show
```
