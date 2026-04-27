# Audio Design Rules · huashu-design

> The audio recipe for all animation demos. Pair with `sfx-library.md` (the asset catalog).
> Battle-tested: huashu-design hero v1–v9 iterations · deep Gemini breakdown of three official Anthropic pieces · 8000+ A/B comparisons

---

## Core Principle · Dual-Track Audio (Iron Rule)

Animation audio **must be designed as two independent layers** — never just one:

| Layer | Role | Time scale | Relationship to visuals | Frequency band |
|---|---|---|---|---|
| **SFX (beat layer)** | Marks every visual beat | 0.2–2 seconds, brief | **Strong sync** (frame-level alignment) | **High-freq 800Hz+** |
| **BGM (ambient foundation)** | Emotional underpinning, soundscape | Continuous 20–60 seconds | Weak sync (section-level) | **Mid-low freq <4kHz** |

**An animation with only BGM is broken** — the audience subconsciously notices "the visuals are moving but there's no audio response." That's the root source of the cheap feel.

---

## Gold Standard · Golden Parameters

These values are hard engineering parameters derived from testing the three official Anthropic pieces + our own v9 final — apply directly:

### Volume
- **BGM volume**: `0.40–0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness gap**: BGM peak **-6 to -8 dB below SFX peak** (not about SFX absolute loudness — it's about the gap)
- **amix parameter**: `normalize=0` (never use `normalize=1` — it flattens dynamic range)

### Frequency Band Isolation (P1 Hard Optimization)
Anthropic's secret is not "SFX volume is louder" — it's **frequency band separation**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # BGM constrained to <4kHz mid-low frequencies
[sfx_raw]highpass=f=800[sfx]      # SFX pushed to 800Hz+ mid-high frequencies
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: the human ear is most sensitive in the 2–5kHz range (the "presence band"). If SFX all live in this band while BGM covers the full spectrum, **SFX gets masked by BGM's high-frequency content**. Using highpass to push SFX up + lowpass to push BGM down gives each its own spectral territory, and SFX clarity jumps a full level.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s — prevents hard cut)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail — sense of closure)
- SFX has its own envelope built in — no additional fade needed

---

## SFX Cue Design Rules

### Density (SFX per 10 seconds)
Real-world SFX density from the three Anthropic pieces has three tiers:

| Piece | SFX per 10s | Product personality | Scenario |
|---|---|---|---|
| Artifacts (ref-1) | **~9 per 10s** | Feature-dense, information-heavy | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure ambient, meditative | Developer tool focus state |
| Word (ref-3) | **~4 per 10s** | Balanced, office rhythm | Productivity tool |

**Heuristic:**
- Calm / focused product personality → low SFX density (0–3 per 10s); BGM-driven
- Energetic / information-dense product personality → high SFX density (6–9 per 10s); SFX drives rhythm
- **Don't fill every visual beat** — restraint is more sophisticated than density. **Deleting 30–50% of cues makes the remaining ones more dramatic.**

### Cue Selection Priority
Not every visual beat needs SFX. Use this priority:

**P0 Required** (omitting creates a jarring gap):
- Typing (terminal / input field)
- Click / selection (user decision moment)
- Focus switch (visual lead transfer)
- Logo reveal (brand convergence)

**P1 Recommended:**
- Element entrance / exit (modal / card)
- Completion / success feedback
- AI generation start / end
- Major transition (scene switch)

**P2 Optional** (too many gets noisy):
- Hover / focus-in
- Progress tick
- Decorative ambient

### Timestamp Alignment Precision
- **Same-frame alignment** (0ms offset): click / focus switch / logo landing
- **1–2 frames early** (-33ms): fast whoosh (gives audience psychological anticipation)
- **1–2 frames late** (+33ms): object landing / impact (matches real-world physics)

---

## BGM Selection Decision Tree

The huashu-design skill includes 6 BGM tracks (`assets/bgm-*.mp3`):

```
What is the animation's personality?
├─ Product launch / tech demo      → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial / tool walkthrough     → bgm-tutorial.mp3 (warm, instructional)
├─ Education / principle explainer → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing / brand promotion     → bgm-ad.mp3 (upbeat, promotional)
└─ Need a variant of any of these  → bgm-*-alt.mp3 (alternate version of each)
```

### Scenarios to Consider No BGM
Reference: Anthropic Code Desktop (ref-2) — **0 SFX + pure Lo-fi BGM** can also be premium.

**When to choose no BGM:**
- Animation duration <10s (not enough time for BGM to establish)
- Product personality is "focus / meditative"
- Scene already has ambient sound / voiceover
- SFX density is very high (to avoid auditory overload)

---

## Scene Recipes (Drop-In Ready)

### Recipe A · Product Launch Hero (huashu-design v9 style)
```
Duration: 25 seconds
BGM: bgm-tech.mp3 · 45% · band <4kHz
SFX density: ~6 per 10s

Cues:
  Terminal typing → type × 4 (0.6s apart)
  Enter key       → enter
  Cards converging → card × 4 (staggered 0.2s)
  Selection       → click
  Ripple          → whoosh
  4 focus switches → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool Feature Demo (reference: Anthropic Code Desktop)
```
Duration: 30–45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0–2 per 10s (minimal)

Strategy: let BGM + explanatory voiceover drive; SFX only at decisive moments
(file saved / command completed)
```

### Recipe C · AI Generation Demo
```
Duration: 15–20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 per 10s (high density)

Cues:
  User input         → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete → feedback/complete-done
  Result appears     → magic/sparkle

Highlight: ai-process can loop 2–3 times throughout the generation process
```

### Recipe D · Pure Ambient Long Take (reference: Artifacts)
```
Duration: 10–15 seconds
BGM: none
SFX: 3–5 carefully crafted cues used standalone

Strategy: each SFX is a lead actor — no BGM muddying things together.
Best for: single-product slow shot, close-up showcase
```

---

## ffmpeg Mixing Templates

### Template 1 · Overlay Single SFX onto Video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX Timeline Composition (aligned to cue times)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters:**
- `adelay=N|N`: first value is left channel delay (ms), second is right channel — write both to ensure stereo alignment
- `normalize=0`: preserve dynamic range — critical
- `-t 25`: clip to specified duration

### Template 3 · Video + SFX Track + BGM (with frequency band isolation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Quick Reference

| Symptom | Root cause | Fix |
|---|---|---|
| SFX inaudible | BGM high-frequency content masking | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| Sound effect too loud / harsh | SFX absolute volume too high | Lower SFX to 0.7, also lower BGM to 0.3, preserve the gap |
| BGM and SFX rhythms conflict | Wrong BGM (used music with a strong beat) | Switch to ambient / minimal synth BGM |
| BGM cuts abruptly at animation end | No fade out | `afade=out:st=N-1.5:d=1.5` |
| SFX overlapping into mud | Too many cues + each SFX too long | Keep SFX duration under 0.5s; cue interval ≥ 0.2s |
| WeChat MP4 has no sound | WeChat sometimes mutes auto-play | Don't worry — user gets sound when they tap; GIFs don't have sound inherently |

---

## Audio-Visual Pairing (Advanced)

### SFX Timbre Should Match Visual Style
- Warm cream / paper-texture visuals → SFX using **wooden / soft** timbres (Morse, paper snap, soft click)
- Cool dark-tech visuals → SFX using **metallic / digital** timbres (beep, pulse, glitch)
- Hand-drawn / playful visuals → SFX using **cartoon / exaggerated** timbres (boing, pop, zap)

Current `apple-gallery-showcase.md` warm cream background → pairs with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass)

### SFX Can Lead the Visual Rhythm
Advanced technique: **design the SFX timeline first, then adjust visual animations to align with SFX** (not the other way around).
Because each SFX cue is a "clock tick," visual animations adapted to SFX rhythm are very stable — conversely, SFX chasing visuals often can't land within ±1 frame, creating a jarring gap.

---

## Quality Check Checklist (Pre-Delivery)

- [ ] Loudness gap: SFX peak − BGM peak = -6 to -8 dB?
- [ ] Frequency bands: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (dynamic range preserved)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] SFX count appropriate for the scene personality (low / medium / high density)?
- [ ] Each SFX aligned with its visual beat (within ±1 frame)?
- [ ] Logo reveal sound long enough (recommend 1.5s)?
- [ ] Mute BGM and listen: does SFX alone have enough rhythmic feel?
- [ ] Mute SFX and listen: does BGM alone have emotional arc?

Each layer should be self-contained when listened to alone. If it only sounds good with both layers combined, the work isn't done.

---

## References

- SFX asset catalog: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Anthropic three-piece deep audio analysis: `AUDIO-BEST-PRACTICES.md` in the project's reference-animations directory
- huashu-design v9 real-world case: `hero-animation-v9-final.mp4` in the project's assets directory
