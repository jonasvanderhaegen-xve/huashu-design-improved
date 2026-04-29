# Dependencies

Runtime requirements for the scripts in this repo. The skill itself (SKILL.md) has no install requirements — only the export and verification scripts do.

## Quick check

```bash
node --version       # need 18+
python --version     # need 3.9+
ffmpeg -version      # need 4.x+
npx playwright --version
```

---

## Required by script

| Script | Requires |
|--------|---------|
| `scripts/render-video.js` | Node 18+, Playwright + Chromium |
| `scripts/export_deck_pdf.mjs` | Node 18+, Playwright + Chromium, `pdf-lib` |
| `scripts/export_deck_stage_pdf.mjs` | Node 18+, Playwright + Chromium |
| `scripts/export_deck_pptx.mjs` | Node 18+, Playwright + Chromium, `pptxgenjs`, `sharp` |
| `scripts/html2pptx.js` | Node 18+, Playwright + Chromium, `sharp` (called by export_deck_pptx.mjs) |
| `scripts/add-music.sh` | ffmpeg 4.x+ (includes ffprobe) |
| `scripts/convert-formats.sh` | ffmpeg 4.x+ |
| `scripts/verify.py` | Python 3.9+, Playwright for Python |

---

## Install instructions

### Node 18+
Download from https://nodejs.org or use a version manager:
```bash
nvm install 18 && nvm use 18   # nvm
# or
fnm install 18 && fnm use 18   # fnm (faster)
```

### Node dependencies (for export scripts)
Run once from the repo root:
```bash
npm install playwright pptxgenjs pdf-lib sharp
npx playwright install chromium
```

### ffmpeg 4.x+
- **macOS**: `brew install ffmpeg`
- **Ubuntu/Debian**: `sudo apt install ffmpeg`
- **Windows**: Download from https://ffmpeg.org/download.html and add to PATH

Verify: `ffmpeg -version` and `ffprobe -version` should both print version info.

### Python 3.9+ + Playwright
```bash
pip install playwright
playwright install chromium
```

---

## Note on `test-prompts.json`

`test-prompts.json` in the repo root is a **manual QA benchmark** — a set of 6 representative prompts covering the skill's major code paths (SaaS design, pitch deck, animation, iOS prototype). Use it to verify the skill behaves correctly after changes:

1. Open a new Claude Code session with the skill loaded
2. Send each prompt from the JSON one by one
3. Check your output against the `expected` field (what the agent should produce) and `tests` field (which behaviors are exercised)

No test runner is needed — this is a human-in-the-loop benchmark, not automated CI.
