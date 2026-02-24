# Design Playground

Visual design configuration playground for UI components — buttons, cards, inputs, layouts, typography, and color schemes. Users tweak visual parameters interactively and copy a natural-language design specification back into OpenCode.

## Controls

Organize controls into collapsible groups. Show the most impactful controls first.

### Shape & Spacing group

- **Border radius** (slider) — 0–32px, default 8px, step 1. Show current value.
- **Padding horizontal** (slider) — 4–48px, default 16px, step 2.
- **Padding vertical** (slider) — 2–32px, default 10px, step 2.
- **Border width** (slider) — 0–4px, default 1px, step 1.
- **Border style** (dropdown) — solid | dashed | dotted | none. Default: solid.

### Color group

- **Background color** (color input or HSL sliders) — default: `#3b82f6` (blue-500).
- **Text color** (color input) — default: `#ffffff`.
- **Border color** (color input) — default: `#2563eb` (blue-600).
- **Background hover color** (color input) — default: `#2563eb`.

### Typography group

- **Font family** (dropdown) — System (sans-serif) | Monospace | Serif. Default: System.
- **Font size** (slider) — 10–28px, default 16px, step 1.
- **Font weight** (dropdown) — 300 (Light) | 400 (Normal) | 500 (Medium) | 600 (Semibold) | 700 (Bold). Default: 500.
- **Text transform** (dropdown) — none | uppercase | lowercase | capitalize. Default: none.
- **Letter spacing** (slider) — -1–4px, default 0px, step 0.5.

### Effects group (collapsible, starts collapsed)

- **Shadow** (dropdown) — none | subtle | medium | pronounced. Default: none. Map to box-shadow values:
  - none: `none`
  - subtle: `0 1px 3px rgba(0,0,0,0.2)`
  - medium: `0 4px 12px rgba(0,0,0,0.3)`
  - pronounced: `0 8px 24px rgba(0,0,0,0.4)`
- **Opacity** (slider) — 0.1–1.0, default 1.0, step 0.05.
- **Transition duration** (slider) — 0–500ms, default 150ms, step 25.
- **Cursor** (dropdown) — pointer | default | not-allowed. Default: pointer.

## Preview

Render a **button element** in the center of the preview panel using inline styles derived from the current state. The button text should be "Click me" or a user-configurable label.

Show the button in three states stacked vertically with labels:

1. **Default** — normal appearance
2. **Hover** — simulated with the hover color applied (no actual hover needed; just show both states)
3. **Disabled** — reduced opacity (0.5), `not-allowed` cursor

Below the three states, show a **CSS output** block (monospace, small font) displaying the computed CSS properties:

```css
.button {
  border-radius: 8px;
  padding: 10px 16px;
  /* ... only non-default properties */
}
```

## Prompt rules

Generate a CSS prompt that an agent can directly apply. Only output properties that differ from defaults.

Structure:

1. If any values changed, output: "Apply these CSS properties to the button component:" followed by a `.button { ... }` CSS block with only the changed properties
2. Use exact CSS values — `border-radius: 12px`, `padding: 14px 28px`, `background: #1e40af` — not qualitative descriptions
3. If all defaults, output: "Default button styling — no changes specified."

Example prompt output:
```
Apply these CSS properties to the button component:

.button {
  border-radius: 12px;
  padding: 14px 28px;
  background: #1e40af;
  color: #ffffff;
  font-weight: 700;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  transition: all 300ms ease;
}
```

## Presets

- **Minimal** — Flat, no shadow, small radius, thin border: `{ borderRadius: 2, shadow: 'none', borderWidth: 1, fontSize: 14, fontWeight: 400, padding: '8px 12px' }`
- **Rounded** — Pill-shaped, subtle shadow, no border: `{ borderRadius: 9999, shadow: 'subtle', borderWidth: 0, fontSize: 16, fontWeight: 500 }`
- **Bold** — Large, pronounced shadow, heavy weight: `{ borderRadius: 8, shadow: 'pronounced', fontSize: 20, fontWeight: 700, padding: '14px 28px' }`
- **Ghost** — Transparent background, visible border, text-colored: `{ background: 'transparent', textColor: '#3b82f6', borderColor: '#3b82f6', borderWidth: 2, shadow: 'none' }`
- **Corporate** — Subtle, professional, medium everything: `{ borderRadius: 4, shadow: 'medium', fontSize: 14, fontWeight: 600, textTransform: 'uppercase', letterSpacing: 1.5 }`

## Mistakes to avoid

- **Describing visual appearance instead of CSS values** — Output `background: #f97316` not "a warm orange background". The agent cannot see the preview; it needs exact values.
- **Ignoring hover state** — Always show the hover variant in the preview. Users need to see both states.
- **Color sliders without preview swatch** — Show a color swatch next to every color control.
- **Not reflecting CSS output** — The CSS block in the preview must update live and match the visual rendering exactly.
- **Preset doesn't reset all values** — When applying a preset, reset ALL values to defaults first, then apply the preset overrides. Otherwise stale values from previous presets bleed through.
