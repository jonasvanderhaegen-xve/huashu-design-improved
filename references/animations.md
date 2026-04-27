# Animations: Timeline Animation Engine

Read this when building animation / motion design HTML. Covers principles, usage, and typical patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: Container for the entire animation. Automatically provides auto-scale (fit viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: A time segment. A Sprite is only visible between `start` and `end`. Internally, read local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: Read the current global time (seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...**:  Easing functions
- **`interpolate(t, from, to, easing?)`**: Interpolate a value based on t

This pattern is inspired by Remotion / After Effects, but lightweight and zero-dependency.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note the range:** `[0, 0.3]` means the fade-in completes in the first 30% of the sprite's duration — after that, opacity stays at 1.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-Character Typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number Count-Up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Segmented Explanation (typical educational animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: show the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: show the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: show the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* subtitle shown throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Characteristic | Use for |
|--------|------|------|
| `linear` | Constant speed | Scrolling captions, continuous animations |
| `easeIn` | Slow→fast | Exit / disappear |
| `easeOut` | Fast→slow | Entrance / appear |
| `easeInOut` | Slow→fast→slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Anthropic-grade primary easing** (physical weight feel) |
| **`overshoot`** ⭐ | **Elastic overshoot** | **Toggle / button pop / emphasis interactions** |
| `spring` | Spring | Interaction feedback, geometry returning to position |
| `anticipation` | Reverse then forward | Emphasizing an action |

**Default primary easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Entrance uses `expoOut`, exit uses `easeIn`, toggle uses `overshoot` — the foundational pattern for Anthropic-grade animation.

## Rhythm and Duration Guide

### Micro-interactions (0.1–0.3s)
- Button hover
- Card expand
- Tooltip appear

### UI Transitions (0.3–0.8s)
- Page switch
- Modal appear
- List item added

### Narrative Animation (2–10s per segment)
- A phase of concept explanation
- Data chart reveal
- Scene transition

### Single narrative segment: maximum 10 seconds
Human attention is limited. 10 seconds per thing — then move to the next.

## Thinking Order for Designing Animation

### 1. Content / story first, animation second

**Wrong:** Starting with a desire for fancy animation, then stuffing in content
**Right:** First clarify what information to convey, then use animation to serve that information

Animation is **signal**, not **decoration**. A fade-in says "this matters, look here" — if everything fades in, the signal is void.

### 2. Write the timeline scene by scene

```
0:00 - 0:03   problem appears (fade in)
0:03 - 0:06   problem magnifies / expands (zoom+pan)
0:06 - 0:09   solution appears (slide in from right)
0:09 - 0:12   solution expands with explanation (typewriter)
0:12 - 0:15   result demo (counter up + chart reveal)
0:15 - 0:18   one-line summary (static, 3 seconds to read)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then the components.

### 3. Assets first

Get images / icons / fonts ready **before** starting. Don't go looking for assets mid-animation — it breaks the rhythm.

## Common Issues

**Choppy animation**
→ Mainly layout thrashing. Use `transform` and `opacity` — don't animate `top` / `left` / `width` / `height` / `margin`. The browser GPU-accelerates `transform`.

**Animation too fast to read**
→ Reading a word takes 100–300ms; a sentence needs 500ms+. If you're telling a story with text, allow at least 3 seconds per sentence.

**Animation too slow, audience bored**
→ Keep interesting visual changes dense. A static frame for more than 5 seconds gets dull.

**Multiple animations interfering with each other**
→ Use CSS `will-change: transform` to tell the browser in advance that the element will move — reduces reflow.

**Recording to video**
→ Use the skill's built-in toolchain (one command, three formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Want more precise frame rendering? Make render(t) a pure function — see `animation-pitfalls.md` item 5

## Working with Video Tools

This skill produces **HTML animations** (running in the browser). If the final output needs to serve as video material:

- **Short animation / concept demo:** Use this method to build HTML animation → screen record
- **Long video / narrative:** This skill focuses on HTML animation — for long-form video use an AI video generation skill or professional video software
- **Motion graphics:** Professional After Effects / Motion Canvas is better suited

## On Libraries Like Popmotion

If you genuinely need physics-based animation (spring, decay, keyframes with precise timing), our engine won't cut it — fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It's sufficient for 90% of cases.
