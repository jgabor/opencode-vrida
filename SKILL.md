---
name: vrida
description: >
  Builds interactive HTML rigs — one file, zero dependencies, batteries included.
  Tweak controls, watch the preview update, copy the prompt. Visual tinkering, assembled to order.
  Activate when the user asks for a rig, explorer, or interactive visual tool.
---

# Vrida — Visual Rig for Interactive Design Adjustment

Vrida is a rig factory. You provide the idea, Vrida assembles a single HTML file — controls on the left, live preview on the right, copyable prompt at the bottom. The user fiddles with knobs and sliders, the preview reacts instantly, and when things look right they hit Copy and paste the result back into the conversation. One file. No dependencies. No allen key required.

## Activation

Listen for the user asking to build a rig, explorer, or interactive visual tool. Keywords: _rig_, _explorer_, _interactive tool_, _visual builder_, _vrida_.

Wait for an explicit request — don't surprise people with rigs they didn't ask for.

## Assembly instructions

### Step 1 — Permission handshake (one-time)

Before reading Vrida skill files, confirm permanent read access for the installed skill directory under `~/.config/opencode/skills/vrida/`.

If access is not already permanently allowed, ask once:

> "Before I continue, do you want me to permanently allow read access for Vrida's `SKILL.md`, `templates/`, and `samples/` files by adding them to your OpenCode allow list?"

If the user says yes, merge this single `read` rule into `~/.config/opencode/opencode.json` without removing existing entries:

```json
{
  "permission": {
    "read": {
      "~/.config/opencode/skills/vrida/**": "allow"
    }
  }
}
```

OpenCode supports `~` home-directory expansion in permission patterns. Use the `~/.config/opencode/...` form in instructions and examples to match OpenCode docs.

If the user says no, continue using normal prompt-time permission requests and do not edit config.

Never add `git` checkout paths to this allow list unless the user explicitly asks for development-mode behavior.

### Step 2 — Pick the right template

Scan `templates/` for `.md` files. Match the user's request to the closest template:

| Template | What it's for |
| --- | --- |
| `templates/canvas.md` | Visual design — components, layouts, spacing, color, typography |
| `templates/graph.md` | Architecture diagrams — components, data flow, layer maps |
| `templates/review.md` | Document review — approve / reject / comment workflow |
| `templates/brainstorm.md` | Feature brainstorming — structured ideation with constraints and metrics |

The user may have added their own templates too — always check the full directory. If nothing fits perfectly, grab the nearest one and improvise.

### Step 3 — Build the rig

Follow the template's instructions. The template defines controls, preview behavior, prompt format, and presets. Vrida's job is to assemble all of that into a working HTML file.

### Step 4 — Deliver it

Save the file to `./vrida/<descriptive-kebab-name>.html` and open it (details in [Getting it to the user](#getting-it-to-the-user) below).

## The ground rules

These apply to every rig, no exceptions. Think of them as the warranty conditions.

**One file, zero dependencies.** Everything — HTML, CSS, JS — lives in a single `.html` file. No CDN links, no imports, no fetches. If the internet goes down, the rig keeps working.

**Instant feedback.** Every control change updates the preview and the prompt immediately. No "Apply" buttons. No lag. The user moves a slider — the preview moves with it.

**Useful prompts, not data dumps.** The prompt must be actionable and self-contained — the agent that receives it has no access to the browser. Output concrete implementation values (CSS properties, specs, tokens) rather than visual descriptions. Frame changes as instructions, not observations: `Apply border-radius: 12px` not `The corners look rounded`. Never output raw key-value pairs like `borderRadius: 12, shadow: true` and never describe what is visually visible without providing the values needed to reproduce it.

**Copy that actually works.** A copy button grabs the prompt text, shows "Copied!" for ~1.5 seconds, then resets. Include a `document.execCommand('copy')` fallback for environments where `navigator.clipboard` isn't available.

**Ready out of the box.** The rig must look polished on first load. Provide 3–5 named presets — cohesive combinations that show it at its best. Never ship a blank or broken initial state.

**Dark, clean, focused.** System font stack for UI. Monospace for code and values. Minimal chrome. Color palette in the `#1a1a2e` / `#16213e` / `#0f3460` range, with `#e94560` or similar as the accent.

**Works on small screens too.** Side-by-side panels on desktop (≥641px). On mobile (≤640px), panels stack and a sticky tab bar at the bottom lets the user flip between Controls, Preview, and Prompt. Details in [Small screen handling](#small-screen-handling).

## The blueprint

Every rig follows the same spatial arrangement:

```text
┌─────────────────────────────────────────────┐
│  Title                    [Preset buttons]  │
├──────────────────┬──────────────────────────┤
│                  │                          │
│   Controls       │      Live Preview        │
│   (~30-35%)      │      (remaining)         │
│                  │                          │
├──────────────────┴──────────────────────────┤
│  Prompt output                       [Copy] │
└─────────────────────────────────────────────┘
```

The controls panel scrolls if it overflows. The prompt panel sits at the bottom, max 200px tall with overflow scroll.

## Small screen handling

Side-by-side layouts collapse into unreadable slivers on a 390px screen. Instead, show one panel at a time with a sticky bottom tab bar:

```text
┌──────────────────────────────┐
│  Title             [Presets] │
├──────────────────────────────┤
│                              │
│   Active panel (scrollable)  │
│                              │
├──────────────────────────────┤
│  [Controls] [Preview] [Prompt] │  ← sticky bottom
└──────────────────────────────┘
```

#### The tab bar (hidden on desktop, visible ≤640px)

```html
<div class="panel-tabs" id="panel-tabs">
  <button class="panel-tab active" data-panel="controls" onclick="showPanel('controls')">Controls</button>
  <button class="panel-tab" data-panel="preview" onclick="showPanel('preview')">Preview</button>
  <button class="panel-tab" data-panel="prompt" onclick="showPanel('prompt')">Prompt</button>
</div>
```

#### Panel structure

Give each section `class="panel"` and a unique `id`. Active panels need `padding-bottom: 90px` so the tab bar doesn't eat the content.

```html
<div class="main">
  <div class="controls panel active" id="panel-controls">…</div>
  <div class="preview panel" id="panel-preview">…</div>
</div>

<div class="prompt-bar panel" id="panel-prompt">
  <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
    <span style="font-size:12px;color:var(--text-dim)">Generated prompt</span>
    <button id="copy-btn" onclick="copyPrompt()">Copy</button>
  </div>
  <pre id="prompt-text"></pre>
</div>
```

#### Mobile CSS

```css
.panel-tabs { display: none; }

@media (max-width: 640px) {
  .panel-tabs {
    display: flex; position: fixed; bottom: 0; left: 0; right: 0; height: 64px;
    background: var(--surface); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
    border-top: 1px solid var(--border); z-index: 100;
    padding-bottom: env(safe-area-inset-bottom);
  }
  .panel-tab {
    flex: 1; padding: 8px 4px; background: none; border: none;
    color: var(--text-dim); font-size: 13px; font-weight: 600; cursor: pointer;
    border-top: 2px solid transparent; transition: all 0.2s;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
  }
  .panel-tab.active { color: var(--accent); border-top-color: var(--accent); }

  .main { display: flex; flex-direction: column; flex: 1; height: auto; }
  .panel { display: none !important; }

  .controls.panel.active { display: block !important; flex: 1; overflow-y: auto; padding-bottom: 90px; }
  .preview.panel.active { display: flex !important; flex: 1; padding-bottom: 90px; }

  .prompt-bar.panel { border-top: none; }
  .prompt-bar.panel.active { display: flex !important; flex: 1; max-height: none; flex-direction: column; padding-bottom: 90px; }

  body[data-active-panel="prompt"] .main { display: none; }
}
```

#### Panel switching logic

```javascript
function showPanel(name) {
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.panel-tab').forEach(t => t.classList.remove('active'));

  const panel = document.getElementById('panel-' + name) || document.querySelector('.' + name + '.panel');
  if (panel) panel.classList.add('active');

  const tab = document.querySelector(`.panel-tab[data-panel="${name}"]`);
  if (tab) tab.classList.add('active');

  document.body.setAttribute('data-active-panel', name);
}
```

On mobile, `updateAll()` can auto-switch to the Preview tab after a control change — just make sure the Controls tab is still reachable so the user doesn't get stuck.

## Under the hood

### One source of truth

All rig state lives in a single object. Controls write to it, preview and prompt read from it. No state scattered across DOM attributes.

```javascript
const DEFAULTS = Object.freeze({
  // every configurable value and its starting point
});

let state = { ...DEFAULTS };

function updateAll() {
  renderPreview();   // redraw the visual output
  generatePrompt();  // rebuild the natural-language prompt
}
```

### Presets snap everything into place

Each preset is a partial state object. Applying one resets to defaults first, then layers on the preset values. Every control in the UI must visually sync — if a preset changes a slider value, the slider must move.

```javascript
const PRESETS = {
  'Clean': { radius: 2, shadow: 'none', size: 14 },
  'Soft':  { radius: 12, shadow: 'subtle', size: 16 },
  'Bold':  { radius: 0, shadow: 'heavy', size: 20 },
};

function applyPreset(name) {
  state = { ...DEFAULTS, ...PRESETS[name] };
  syncControlsToState();  // update every slider, dropdown, toggle
  updateAll();
}
```

### Prompt generation

Only mention what the user actually changed. Skip defaults — nobody needs to be told "keep doing what you were doing." For design rigs, output concrete implementation values (CSS properties, tokens, specs) rather than prose descriptions. The agent receiving the prompt has no browser — it cannot see the preview. Give it values it can apply directly.

```javascript
function generatePrompt() {
  const lines = ['.component {'];

  if (state.radius !== DEFAULTS.radius) {
    lines.push(`  border-radius: ${state.radius}px;`);
  }

  const SHADOW_VALUES = { none: 'none', subtle: '0 2px 6px rgba(0,0,0,0.15)', heavy: '0 8px 24px rgba(0,0,0,0.35)' };
  if (state.shadow !== DEFAULTS.shadow) {
    lines.push(`  box-shadow: ${SHADOW_VALUES[state.shadow]};`);
  }

  lines.push('}');

  const output = lines.length > 2
    ? `Apply these CSS changes to the component:\n\n${lines.join('\n')}`
    : 'No changes from defaults.';

  document.getElementById('prompt-text').textContent = output;
}
```

### Clipboard

```javascript
function copyPrompt() {
  const text = document.getElementById('prompt-text').textContent;
  if (navigator.clipboard && navigator.clipboard.writeText) {
    navigator.clipboard.writeText(text).then(() => flashCopied());
  } else {
    // Fallback for non-HTTPS or restricted environments
    const ta = document.createElement('textarea');
    ta.value = text;
    document.body.appendChild(ta);
    ta.select();
    document.execCommand('copy');
    document.body.removeChild(ta);
    flashCopied();
  }
}

function flashCopied() {
  const btn = document.getElementById('copy-btn');
  btn.textContent = 'Copied!';
  setTimeout(() => { btn.textContent = 'Copy'; }, 1500);
}
```

## Getting it to the user

### Where to put it

Save rigs to `./vrida/`:

```bash
mkdir -p ./vrida
```

Use descriptive kebab-case names: `./vrida/button-design.html`, `./vrida/api-architecture.html`, `./vrida/rfc-review.html`.

### First time setup

The first time a rig is created (the `./vrida/` directory didn't exist before), ask:

> "I've created the `./vrida/` directory for your rigs. Want me to add it to `.gitignore`? They're usually throwaway, but you might want to keep them."

If yes, append `vrida/` to `.gitignore` (create the file if needed, skip if the entry already exists). If no, move on quietly.

### Opening it

Try to open it in the user's browser:

```bash
# macOS
open ./vrida/<name>.html

# Linux
xdg-open ./vrida/<name>.html
```

If that works: _"Rig saved to `./vrida/<name>.html` and opened in your browser."_

### Remote / headless fallback

If there's no display server (SSH, containers, headless), spin up a quick HTTP server:

```bash
# Option A — Python
python3 -m http.server 8765 --directory ./vrida/ --bind 0.0.0.0 &

# Option B — Node
npx -y serve ./vrida/ -l 8765 --no-clipboard &
```

Then tell the user the URL with their hostname/IP so they can open it from any device on the network.

If neither Python nor Node is available: _"Rig saved to `./vrida/<name>.html`. Open this file in a browser to use it."_

## Things that will ruin the experience

These are the most common ways a rig goes from delightful to broken. Avoid them.

| Symptom | What went wrong |
| --- | --- |
| Prompt reads like JSON | You dumped state values instead of writing a human sentence |
| Wall of controls overwhelms the user | Group by concern, tuck advanced options into a collapsible `<details>` |
| Preview lags behind controls | A control isn't calling `updateAll()` on change |
| Blank or broken on first load | Missing defaults — every value needs a sensible starting point |
| Rig breaks offline | You added a CDN link or external dependency |
| Prompt is useless without the rig open | Output concrete values (CSS, specs) not visual descriptions — the agent cannot see the preview |
| Copy does nothing on Firefox iOS | Missing `document.execCommand('copy')` fallback |
| Preset changes state but controls don't move | `syncControlsToState()` is missing or incomplete |
| No "Copied!" flash | Users click and wonder if anything happened |
| Presets overflow the header on small screens | Use `overflow-x: auto; flex-wrap: nowrap` |
| Two squished panels on a phone | Use the tab bar pattern instead of forcing side-by-side |
| Prompt bar eats half the mobile screen | A 200px fixed footer is fine on desktop, brutal on mobile — make it a tab |
| Auto-switching to Preview traps the user | If you auto-switch on control change, keep the Controls tab reachable |

## Bring your own template

Vrida discovers templates from the `templates/` directory automatically. To add a new rig type, drop a `.md` file in there following this structure:

```markdown
# Template Name

What this rig type is for.

## Controls

- Control name (type: slider/dropdown/toggle/color/text) — description, default, range

## Preview

What the live preview shows and how it responds to controls.

## Prompt rules

How to generate the prompt:
- What to include when values differ from defaults
- Descriptive language to use
- Sentence structure

## Presets

- **Preset Name** — description: { key: value, ... }

## Mistakes to avoid

Template-specific gotchas.
```

When a user's request is ambiguous, list the available templates and let them pick. When it's clear, auto-select and go.
