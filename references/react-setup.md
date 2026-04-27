# React + Babel Project Specifications

Required technical specs when prototyping with HTML + React + Babel. Violating these will break things.

## Pinned Script Tags (use these exact versions)

Place these three script tags in the HTML `<head>`, using **pinned versions + integrity hashes**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Do not** use unpinned versions like `react@18` or `react@latest` — these cause version drift and caching issues.

**Do not** omit `integrity` — it's your defense if the CDN is hijacked or tampered with.

## File Structure

```
project-name/
├── index.html               # main HTML
├── components.jsx           # component file (loaded with type="text/babel")
├── data.js                  # data file
└── styles.css               # extra CSS (optional)
```

Loading order in HTML:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- finally the main entry -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Do not** use `type="module"` — it conflicts with Babel.

## Three Non-Negotiable Rules

### Rule 1: styles objects must use unique names

**Wrong** (guaranteed to break with multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← same name overwrite!
const styles = { container: {...}, header: {...} };
```

**Correct**: Each component file uses a unique prefix for its styles.

```jsx
// terminal.jsx
const terminalStyles = { 
  screen: {...}, 
  line: {...} 
};

// sidebar.jsx
const sidebarStyles = { 
  container: {...}, 
  item: {...} 
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

This rule is **non-negotiable**. Every time you write `const styles = {...}`, it must be replaced with a specific name — otherwise you get a full-stack error on multi-component load.

### Rule 2: Scope is not shared — export manually

**Key understanding**: Each `<script type="text/babel">` is compiled by Babel independently; their **scopes do not cross**. A `Terminal` component defined in `components.jsx` is **undefined by default** in `pages.jsx`.

**Solution**: At the end of each component file, export the components / utilities you want to share to `window`:

```jsx
// end of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // list everything you need elsewhere
});
```

Then `pages.jsx` can use `<Terminal />` directly, because JSX looks up `window.Terminal`.

### Rule 3: Never use scrollIntoView

`scrollIntoView` pushes the entire HTML container upward, breaking the web harness layout. **Never use it.**

Alternatives:
```js
// scroll to position within container
container.scrollTop = targetElement.offsetTop;

// or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling Claude API (inside HTML)

Some design-agent native environments (e.g. Claude.ai Artifacts) provide a no-config `window.claude.complete`, but most agent environments (Claude Code / Codex / Cursor / Trae / etc.) **don't have it locally**.

If your HTML prototype needs to call an LLM for a demo (e.g. building a chat interface), you have two options:

### Option A: No real call — use mock

Recommended for demo scenarios. Write a fake helper that returns preset responses:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // simulate delay
    return "This is a mock response. Replace with a real API when deploying.";
  }
};
```

### Option B: Real Anthropic API call

Requires an API key; users must paste their own key into the HTML to run it. **Never hardcode the key in the HTML.**

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5-20251001',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: Calling the Anthropic API directly from the browser hits CORS restrictions. If the user's preview environment doesn't support CORS bypass, this path is blocked. Use Option A mock instead, or tell the user they need a proxy backend.

### Option C: Generate mock data using the agent's LLM capability

For local demos only: generate mock response data in the current agent session (using that agent's LLM capability or a multi-model skill the user has installed), then hardcode it into the HTML. The HTML then runs with zero API dependency at runtime.

## Typical HTML Starter Template

Copy this template as the skeleton for React prototypes:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body { 
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- main entry -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ You defined `const styles` in one file and another file overwrote it. Give each a specific prefixed name.

**`Terminal is not defined`**
→ Cross-file reference where scope doesn't cross. At the end of the file that defines `Terminal`, add `Object.assign(window, {Terminal})`.

**Entire page white screen, no console errors**
→ Most likely a JSX syntax error that Babel didn't surface. Temporarily swap `babel.min.js` for the unminified `babel.js` — error messages will be clearer.

**`ReactDOM.createRoot is not a function`**
→ Wrong version. Confirm you're using react-dom@18.3.1 (not 17 or another version).

**`Objects are not valid as a React child`**
→ You rendered an object instead of JSX/string. Usually `{someObj}` should be `{someObj.name}`.

## How to Split Large Projects into Files

**Single files > 1000 lines** are hard to maintain. File-splitting approach:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # primitives: Button, Card, Badge...
│   ├── components.jsx      # business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # home
│   │   ├── detail.jsx      # detail
│   │   └── settings.jsx    # settings
│   ├── router.jsx          # simple router (React state switch)
│   └── app.jsx             # entry component
└── data.js                 # mock data
```

Load in order in HTML:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

**At the end of each file**, use `Object.assign(window, {...})` to export everything that needs to be shared.
