# Animation Pitfalls: Real Bugs and Rules from HTML Animation

The most common bugs when writing animations — and how to avoid them. Every rule here comes from a real failure.

Read this before writing any animation. It will save you an iteration.

## 1. Stacked Layout — `position: relative` Is Always Required

**The bug**: a `sentence-wrap` element contained 3 `bracket-layer` children (`position: absolute`). No `position: relative` on `sentence-wrap`, so the absolute-positioned brackets used `.canvas` as their coordinate origin and ended up 200px off the bottom of the screen.

**Rules:**
- Any container that has `position: absolute` children **must** explicitly set `position: relative`
- Even when no visual offset is needed, `position: relative` is required as the coordinate anchor
- When writing `.parent { ... }` with `.child { position: absolute }` inside, reflexively add `relative` to the parent

**Quick check**: for every `position: absolute`, walk up the ancestor chain and confirm the nearest positioned ancestor is the one *you intend* as the coordinate system.

## 2. Character Trap — Don't Rely on Rare Unicode

**The bug**: wanted to use `␣` (U+2423 OPEN BOX) to visualize a "space token". Noto Serif SC / Cormorant Garamond don't have this glyph — renders as blank / tofu. Audience sees nothing.

**Rules:**
- **Every character that appears in an animation must exist in the selected font**
- Common rare-character blocklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To represent metacharacters like "space / return / tab", use **CSS-constructed semantic boxes**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Validate emoji too: some emoji fall back to gray boxes in fonts other than Noto Emoji — use the `emoji` font-family or SVG

## 3. Data-Driven Grid/Flex Templates

**The bug**: code had `const N = 6` tokens, but CSS hardcoded `grid-template-columns: 80px repeat(5, 1fr)`. The 6th token had no column — entire matrix misaligned.

**Rules:**
- When count comes from a JS array (`TOKENS.length`), CSS templates must also be data-driven
- Option A: inject via CSS variable from JS
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` and let the browser expand automatically
- **Ban the "hardcoded number + JS constant" combination** — when N changes, CSS won't update

## 4. Transition Gap — Scene Switches Must Be Continuous

**The bug**: zoom1 (13–19s) → zoom2 (19.2–23s) — the main sentence was already hidden. zoom1 fade out (0.6s) + zoom2 fade in (0.6s) + stagger delay (0.2s+) = about 1 second of pure blank canvas. Audience thought the animation had frozen.

**Rules:**
- When switching scenes continuously, fade out and fade in must **cross-overlap** — don't wait for the previous scene to fully disappear before starting the next
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s blank in between

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fade out 0.4s early
  if (t >= 18.6) showZoom('zoom2');    // fade in simultaneously (cross-fade)
  ```
- Or use an "anchor element" (e.g. the main sentence) as a visual bridge between scenes — briefly bring it back during the zoom switch
- Calculate CSS transition durations carefully so one transition doesn't fire before the previous one ends

## 5. Pure Render Principle — Animation State Must Be Seekable

**The bug**: chained animation state via `setTimeout` + `fireOnce(key, fn)`. Plays fine normally, but when doing frame-by-frame recording / seeking to an arbitrary time, the prior `setTimeout` calls have already fired and can't be "gone back to."

**Rules:**
- `render(t)` should ideally be a **pure function**: given t, outputs a unique DOM state
- If side effects are necessary (e.g. class toggling), use a `fired` set with explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related `setTimeout` should not span >1 second — on seek-back they'll fire out of order

## 6. Measuring Before Fonts Load = Measuring Wrong

**The bug**: called `charRect(idx)` to measure bracket positions immediately on `DOMContentLoaded` — fonts hadn't loaded yet. Every character width was the fallback font's width, so positions were wrong. After fonts loaded (~500ms later), `left: Xpx` still held the old values — permanently offset.

**Rules:**
- Any layout code that depends on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // fonts ready now — measurements are accurate
      tick();              // start animation
    });
  });
  ```
- The extra `requestAnimationFrame` gives the browser one frame to commit layout
- If using Google Fonts CDN, add `<link rel="preconnect">` to speed up initial load

## 7. Recording Readiness — Build Hooks for Video Export

**The bug**: Playwright `recordVideo` starts recording the moment the context is created. Page load + font load = first 2 seconds captured. Delivered video had 2 seconds of blank / flicker at the start.

**Rules:**
- Use the `render-video.js` tool to handle: warmup navigate → reload to restart animation → wait duration → ffmpeg trim head + convert to H.264 MP4
- **Frame 0** must be the complete initial state with layout already in place (not blank, not loading)
- Want 60fps? Use ffmpeg `minterpolate` in post — don't count on the browser's source frame rate
- Want GIF? Two-pass palette (`palettegen` + `paletteuse`) — can compress a 30s 1080p animation to ~3MB

See `video-export.md` for full script invocation.

## 8. Batch Export — Temp Directories Must Include PID to Prevent Concurrency Conflicts

**The bug**: ran `render-video.js` with 3 processes recording 3 HTMLs in parallel. `TMP_DIR` was named with only `Date.now()`. All 3 started at the same millisecond — shared the same tmp directory. The first to finish cleaned it up. The other two tried to read the directory → `ENOENT` → both crashed.

**Rules:**
- Any temp directory that could be shared across processes **must** include a **PID or random suffix**:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you genuinely need parallel multi-file processing, use shell `&` + `wait` rather than forking inside a node script
- When recording multiple HTMLs in batch, the safe default is **serial** (up to 2 in parallel is fine; 3+ queue them)

## 9. Progress Bars / Replay Buttons in Recordings — Chrome Elements Polluting the Video

**The bug**: animation HTML included a `.progress` bar, `.replay` button, and `.counter` timestamp — useful for human playback debugging. When exported to MP4 for delivery, these appeared at the bottom of the video, as if the developer tools had been captured.

**Rules:**
- Manage "chrome elements" (progress bar / replay button / footer / masthead / counter / phase labels) separately from the video content itself
- **Establish class name convention** `.no-record`: any element with this class is automatically hidden by the recording script
- The script (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Inject via Playwright's `addInitScript` (fires before every navigate — stable through reloads)
- To view the raw HTML (with chrome visible), add a `--keep-chrome` flag

## 10. Animation Repeats at the Start of Recording — Warmup Frame Leak

**The bug**: old `render-video.js` flow was `goto → wait fonts 1.5s → reload → wait duration`. Recording starts when the context is created — the warmup phase had already played part of the animation, then reload restarted from t=0. Result: video started with "animation mid-point → cut → animation from 0" — obvious repeat.

**Rules:**
- **Warmup and record must use separate contexts**:
  - Warmup context (no `recordVideo` option): only loads the URL, waits for fonts, then closes
  - Record context (with `recordVideo`): starts fresh — animation records from t=0
- ffmpeg `-ss trim` can only cut Playwright's small startup latency (~0.3s) — **it cannot cover warmup frames**; the source must be clean
- Recording context closing = webm file flushed to disk (Playwright constraint)
- Corresponding code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't Draw Fake Chrome Inside the Canvas

**The bug**: the animation used a `Stage` component that already has its own scrubber + timecode + pause button (all `.no-record` chrome — auto-hidden on export). I also drew a decorative `"00:60 ──── CLAUDE-DESIGN / ANATOMY"` progress bar at the bottom of the canvas — felt clever at the time. **Result**: the user saw two progress bars — one from Stage controls, one from my decoration. Visual collision, immediately flagged as a bug. "Why is there a progress bar inside the video?"

**Rules:**

- Stage already provides: scrubber + timecode + pause/replay. **Do not draw** progress indicators, current timecodes, copyright strips, or chapter counters inside the canvas — they either collide with Stage chrome or they're filler slop (violates the "earn its place" principle).
- "Magazine page-number feel", "editorial bottom bar", "attribution strip" — these are high-frequency AI filler additions. Every time one appears, ask: does it communicate something irreplaceable? Or is it just filling whitespace?
- If you're convinced a bottom strip genuinely must exist (e.g. the animation's topic is a player UI), it must be **narratively necessary** and **visually distinct from the Stage scrubber** (different position, different form, different tone).

**Element ownership test** (every element drawn on canvas must answer this):

| It belongs to | Treatment |
|---|---|
| A specific scene's narrative content | OK — keep it |
| Global chrome (controls / debugging) | Add `.no-record` class — hidden on export |
| **Neither a scene nor chrome** | **Delete.** It's orphaned content — it's always filler slop |

**Self-check (3 seconds before delivery)**: take a static screenshot and ask:

- Is there anything inside the canvas that looks like a video player UI (horizontal progress bar, timecode, control button shapes)?
- If yes — does removing it hurt the narrative? If not, remove it.
- Is the same type of information (progress / time / attribution) appearing twice? Consolidate it into chrome, one place.

**Anti-examples**: bottom of canvas shows `00:42 ──── PROJECT NAME`; bottom-right shows "CH 03 / 06" chapter counter; canvas edge shows version number "v0.3.1" — all fake-chrome filler.

## 12. Recording Lead-in Blank + Recording Start Offset — The `__ready` × tick × lastTick Triple Trap

**Bug A (lead-in blank)**: exporting a 60-second animation to MP4, the first 2–3 seconds are blank. `ffmpeg --trim=0.3` can't cut it.

**Bug B (start offset — real incident 2026-04-20)**: exporting a 24-second video, user perceived "video only starts playing at 19 seconds." What actually happened: animation started recording at t=5, recorded to t=24, then looped back to t=0, then recorded another 5 seconds to end — so the last 5 seconds of the video were the animation's true beginning.

**Root cause** (shared by both bugs):

Playwright `recordVideo` starts writing WebM the moment `newContext()` is called. By the time Babel/React/fonts load, L seconds (2–6s) have elapsed. The recording script waits for `window.__ready = true` as the "animation starts here" anchor — and it must be strictly paired with animation `time = 0`. Two common failure patterns:

| Failure | Symptom |
|---|---|
| `__ready` set in `useEffect` or synchronous setup (before the first tick frame) | Recording script thinks animation has started; WebM is still recording a blank page → **lead-in blank** |
| tick's `lastTick = performance.now()` initialized at **script top level** | Font load time L is counted into the first frame's `dt`, causing `time` to jump to L immediately → recording is offset by L seconds throughout → **start offset** |

**✅ Correct starter tick template** (hand-written animations must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ don't play by default — wait for fonts ready
let lastTick = null;   // ❗ sentinel — first tick frame forces dt=0 (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ pair: recording start and animation t=0 are the same frame
    render(0);               // re-render to ensure DOM is ready (fonts are loaded at this point)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only advances after the first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // no loop during recording; 0.001s preserves the final frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// don't rAF at top level — wait for fonts before starting
document.fonts.ready.then(() => {
  render(0);                 // draw the initial frame first (fonts are ready)
  playing = true;
  requestAnimationFrame(tick);  // first tick pairs __ready with t=0
});

// ━━━━━━ seek interface (used by render-video for defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct:**

| Step | Why it must be this way |
|---|---|
| `lastTick = null` + first frame `return` | Prevents the L seconds from "script load to first tick execution" from counting as animation time |
| `playing = false` default | During font load, `tick` can run without advancing `time` — no render misalignment |
| `__ready` set on the first tick frame | Recording script starts its timer at this moment, which matches the canvas showing animation t=0 |
| Tick started inside `document.fonts.ready.then(...)` | Avoids fallback font width measurements and prevents first-frame font jump |
| `window.__seek` exposed | Lets `render-video.js` actively correct — second line of defense |

**Corresponding defenses in the recording script:**
1. Inject `window.__recording = true` via `addInitScript` (before page goto)
2. `waitForFunction(() => window.__ready === true)` — record the offset for ffmpeg trim
3. **Also**: after `__ready`, actively call `page.evaluate(() => window.__seek && window.__seek(0))` to force-reset any time drift in the HTML — second line of defense for HTMLs that don't strictly follow the starter template

**Verification**: after exporting MP4:
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
Frame 0 must be the animation's initial state at t=0 (not mid-animation, not black). Last frame must be the animation's final state (not a loop restart).

**Reference implementation**: the `Stage` component in `assets/animations.jsx` and `scripts/render-video.js` both implement this protocol. Hand-written HTML must use the starter tick template — every line defends against a specific real bug.

## 13. Disable Loop During Recording — `window.__recording` Signal

**The bug**: animation `Stage` defaults to `loop=true` (convenient for browser preview). `render-video.js` waited an extra 300ms after the duration as a buffer before stopping. Those 300ms let Stage enter the next loop cycle. ffmpeg `-t DURATION` cut the export — the last 0.5–1s fell into the next loop iteration. Video ending suddenly jumped back to the first frame (Scene 1). Audience thought the video was broken.

**Root cause**: no handshake between the recording script and the HTML. HTML doesn't know it's being recorded — it loops as normal.

**Rules:**

1. **Recording script**: inject `window.__recording = true` via `addInitScript` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: detect this signal and force `loop=false`:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ 0.001 keeps final-frame Sprites from being hidden
   ```

3. **Final Sprite's `fadeOut`**: should be set to `fadeOut={0}` in recording mode — otherwise the video end fades to transparent/dark. Users expect to pause on a clean final frame, not a fade-out. For hand-written HTML, always set `fadeOut={0}` on the ending Sprite.

**Reference implementation**: `assets/animations.jsx` Stage / `scripts/render-video.js` both have the handshake built in. Hand-written Stage must implement `__recording` detection — otherwise recording will always hit this bug.

**Verification**: after exporting MP4, run `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png` and confirm the last 0.2 seconds are still the expected final frame — no sudden cut to another scene.

## 14. 60fps Video Defaults to Frame Duplication — minterpolate Has Compatibility Issues

**The bug**: `convert-formats.sh` used `minterpolate=fps=60:mi_mode=mci...` to generate 60fps MP4. On some versions of macOS QuickTime / Safari, the file wouldn't open (black screen or refused to load). VLC / Chrome worked fine.

**Root cause**: minterpolate produces H.264 elementary streams with SEI / SPS fields that some players can't parse.

**Rules:**

- Default 60fps uses simple `fps=60` filter (frame duplication) — broad compatibility (QuickTime / Safari / Chrome / VLC all work)
- High-quality interpolation uses `--minterpolate` flag — opt-in, but **must be tested on the target player before delivery**
- The value of the 60fps label is **platform algorithm recognition** (Bilibili / YouTube prioritize 60fps streams) — perceived smoothness gain for CSS animations is minimal
- Add `-profile:v high -level 4.0` to improve H.264 universal compatibility

**`convert-formats.sh` now defaults to compatibility mode.** To opt into high-quality interpolation:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + External `.jsx` CORS Trap — Single-File Delivery Must Inline the Engine

**The bug**: animation HTML used `<script type="text/babel" src="animations.jsx"></script>` to load the engine externally. Opening locally by double-click (`file://` protocol) → Babel Standalone uses XHR to fetch `.jsx` → Chrome throws `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → entire page black screen. Doesn't trigger `pageerror` — only a console error — easy to misdiagnose as "animation not triggering."

Starting an HTTP server doesn't always fix it either — on machines with a global proxy, `localhost` also routes through the proxy and returns 502 / connection failure.

**Rules:**

- **Single-file delivery (HTML that works by double-click)** → `animations.jsx` must be **inlined** inside a `<script type="text/babel">...</script>` tag — do not use `src="animations.jsx"`
- **Multi-file project (demonstrated via HTTP server)** → external loading is fine, but write the startup command explicitly: `python3 -m http.server 8000`
- The deciding question: is the deliverable "an HTML file" or "a project directory with a server"? The former requires inlining
- Stage / animations.jsx is often 200+ lines — pasting it into an HTML `<script>` block is completely acceptable; don't worry about size

**Minimum verification**: double-click your generated HTML — **not via any server**. If Stage displays the animation's first frame correctly, it passes.

## 16. Cross-Scene Color Visibility — Don't Hardcode Colors on Elements Shared Across Scenes

**The bug**: in a multi-scene animation, `ChapterLabel` / `SceneNumber` / `Watermark` and other **elements that appear across all scenes** had `color: '#1A1A1A'` (dark text) hardcoded in the component. Fine for the first 4 light-background scenes. On scene 5 with a black background, "05" and the watermark simply vanished — no error, no triggered check, critical information invisible.

**Rules:**

- **Canvas elements reused across multiple scenes** (chapter labels / scene numbers / timecodes / watermarks / copyright strips) **must not hardcode color values**
- Use one of three approaches:
  1. **`currentColor` inheritance**: element only sets `color: currentColor`; the parent scene container sets the computed value
  2. **invert prop**: component accepts `<ChapterLabel invert />` to manually switch between light and dark
  3. **Automatic calculation from background**: `color: contrast-color(var(--scene-bg))` (CSS 4 new API, or JS logic)
- Before delivery, use Playwright to capture **a representative frame from each scene** and visually confirm all cross-scene elements are visible

This pitfall is insidious — **no error fires**. Only human eyes or OCR can catch it.

## Quick Pre-Flight Checklist (5 seconds before starting)

- [ ] Every `position: absolute` element has a `position: relative` parent?
- [ ] Every special character in the animation (`␣` `⌘` emoji) exists in the selected font?
- [ ] Grid/Flex column count matches the JS data length?
- [ ] Scene transitions use cross-fades — no >0.3s blank gaps?
- [ ] DOM measurement code is wrapped in `document.fonts.ready.then()`?
- [ ] `render(t)` is pure, or has an explicit reset mechanism?
- [ ] Frame 0 is the complete initial state — not blank?
- [ ] No fake chrome inside the canvas (progress bar / timecode / bottom attribution strip colliding with Stage scrubber)?
- [ ] Animation tick sets `window.__ready = true` on the first frame? (built into animations.jsx; hand-written HTML must add it)
- [ ] Stage detects `window.__recording` and forces `loop=false`? (required for hand-written HTML)
- [ ] Final Sprite has `fadeOut={0}` set (video ends on a clean frame)?
- [ ] 60fps MP4 uses frame-duplication mode by default (compatibility); high-quality interpolation only with `--minterpolate`?
- [ ] After export: extracted frame 0 and final frame and confirmed they're the animation's initial / final states?
- [ ] Task involves a specific brand (Stripe / Anthropic / Lovart / ...): completed the Brand Asset Protocol (SKILL.md §1.a five steps)? `brand-spec.md` written?
- [ ] Single-file HTML deliverable: `animations.jsx` is inlined — not `src="..."`? (external .jsx causes CORS black screen under `file://`)
- [ ] Elements appearing across scenes (chapter labels / watermarks / scene numbers) have no hardcoded colors? Visible on every scene background?
