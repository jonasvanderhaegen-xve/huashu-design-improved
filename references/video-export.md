# Video Export: HTML Animations to MP4 / GIF

Once the animation HTML is done, users often ask "can this be exported to video?" This guide covers the complete workflow.

## When to Export

**Export timing:**
- Animation fully working and visually verified (Playwright screenshots confirm correct state at each time point)
- User has seen it in the browser at least once and confirmed it looks good
- **Do not** export while animation bugs remain unresolved — changes are far more expensive after video export

**User trigger phrases:**
- "Can this be exported to video?"
- "Convert to MP4"
- "Make it a GIF"
- "60fps"

## Output Specifications

By default, deliver three formats and let the user choose:

| Format | Specs | Best for | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat embed, Video Channel, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate frames · H.264 · CRF 18 | High-frame-rate showcase, Bilibili, portfolio | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette optimized | Twitter/X, README, Slack preview | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a base 25fps MP4. Requires global playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional parameters:
- `--duration=30` animation duration (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to trim from the start (removes reload + font loading time)
- `--fontwait=1.5` font loading wait time (seconds) — increase when using many fonts

Output: same directory as the HTML, same name with `.mp4`.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4. Choose by mood from the built-in BGM library, or supply your own audio. Automatically matches duration and adds fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (at `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Best for |
|-----------|-------|---------|
| `tech` (default) | Apple Silicon / keynote-style, minimal synth + piano | Product launch, AI tools, skill promos |
| `ad` | Upbeat modern electronic with build + drop | Social media ads, product teasers, promo reels |
| `educational` | Warm and bright, light guitar / electric piano, inviting | Science explainers, tutorial intros, course previews |
| `educational-alt` | Same vibe, alternative track | Same as above |
| `tutorial` | Lo-fi ambient, nearly invisible | Software demos, coding tutorials, long walkthroughs |
| `tutorial-alt` | Alternative in same vibe | Same as above |

**Behavior:**
- Music is trimmed to video duration
- 0.3s fade-in + 1s fade-out (avoids hard cuts)
- Video stream `-c:v copy` (no re-encode), audio AAC 192k
- `--music=<path>` takes priority over `--mood` — supply any external audio file directly
- An invalid mood name lists all available options — no silent failures

**Typical pipeline** (animation export trio + music):
```bash
node render-video.js animation.html                        # record screen
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# or for specific scenes:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generates a 60fps version and GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — defaults to `fps=60` frame duplication (widely compatible); add `--minterpolate` for high-quality frame interpolation
- `<name>.gif` — palette-optimized GIF (default 960px wide, configurable)

**60fps Mode Selection:**

| Mode | Command | Compatibility | Use case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime / Safari / Chrome / VLC — all pass | General delivery, platform uploads, social media |
| minterpolate frames | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime / Safari may reject | Bilibili and other platforms requiring true interpolation — **test in target player before delivery** |

Why the default changed to frame duplication: minterpolate's H.264 elementary stream has a known compat bug — the previous minterpolate default caused repeated "macOS QuickTime won't open" incidents. See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) — works on most social platforms
- 1280 — sharper but larger file
- 600 — prioritized loading on Twitter/X

## Complete Workflow (Standard Recommendation)

After the user says "export video":

```bash
cd <project-dir>

# assumes $SKILL points to this skill's root directory (adjust to your install location)

# 1. record base 25fps MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. derive 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# output:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (for Troubleshooting)

### Playwright recordVideo Gotchas

- Frame rate fixed at 25fps — 60fps direct recording is not possible (Chromium headless compositor ceiling)
- Recording starts at context creation — `trim` must be used to remove leading load time
- Default output is WebM — ffmpeg conversion to H.264 MP4 required for universal playback

`render-video.js` already handles all of the above.

### ffmpeg minterpolate Parameters

Current configuration: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable-size block motion compensation

Works well for CSS **transform animations** (translate/scale/rotate).
May produce slight ghosting on **pure fades** — if the user dislikes it, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why the GIF Palette Requires Two Passes

GIF supports only 256 colors. A single-pass GIF compresses the entire animation's color range into a 256-color generic palette — this muddies subtle palettes like cream + orange.

Two passes:
1. `palettegen=stats_mode=diff` — scans the entire animation first to build an **optimal palette for this specific animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encodes using that palette; rectangle diff updates only changed regions, significantly reducing file size

`dither=bayer` produces smoother fade transitions than `none`, but slightly larger files.

## Pre-flight Check (Before Exporting)

30-second self-check before exporting:

- [ ] HTML has run completely in the browser with no console errors
- [ ] Frame 0 of the animation is the complete initial state (not a blank loading screen)
- [ ] The last frame is a stable end state (not mid-transition)
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`)
- [ ] Duration parameter matches actual animation duration in HTML
- [ ] Stage in HTML detects `window.__recording` to force loop=false (mandatory check for hand-written Stage; automatic when using `assets/animations.jsx`)
- [ ] Trailing Sprite has `fadeOut={0}` (last video frame does not fade out)
- [ ] Includes "Created by Huashu-Design" watermark (required for animation scenarios; prefix third-party brand work with "Unofficial · ". See SKILL.md § "Skill Promo Watermark")

## Delivery Handoff Note

Standard format to give the user after export completes:

```
**Full Delivery**

| File | Format | Specs | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolated) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion-estimated frames — works well for transform animations
- GIF uses palette optimization — 30s animations compress to around 3MB

Say the word if you need a different size or frame rate.
```

## Common Follow-up Requests

| User says | Response |
|---|---|
| "Too big" | MP4: raise CRF to 23–28; GIF: reduce resolution to 600px or fps to 10 |
| "GIF looks blurry" | Increase `gif_width` to 1280; or suggest switching to MP4 (WeChat Moments also supports it) |
| "Need vertical 9:16" | Change HTML source to `--width=1080 --height=1920` and re-record |
| "Add watermark" | Add `-vf "drawtext=..."` or an `overlay=` PNG via ffmpeg |
| "Need transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "Need lossless" | Set CRF to 0 + preset veryslow (file will be ~10× larger) |
