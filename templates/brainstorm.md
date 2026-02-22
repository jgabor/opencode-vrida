# Brainstorm

Interactive feature brainstorming playground for exploring and refining a new feature or improvement idea. Users shape their thinking through structured controls — feature name, problem statement, target user, scope, constraints, success metrics — and copy a polished brainstorming brief back into OpenCode.

## Controls

Organize controls into logical groups. Keep the most important inputs (feature name, problem) at the top.

### Feature Identity group

- **Feature name** (text input) — short, descriptive name for the feature. Default: empty. Placeholder: "e.g. Dark mode support".
- **Problem statement** (textarea, 3 rows) — the problem or pain point this feature addresses. Default: empty. Placeholder: "What problem does this solve?"
- **Proposal** (textarea, 3 rows) — high-level description of the proposed solution. Default: empty. Placeholder: "How would this work?"

### Context group

- **Target user** (dropdown) — End User | Developer | Admin | Internal Team | All Users. Default: End User.
- **Priority** (radio buttons, inline) — Low | Medium | High | Critical. Default: Medium.
- **Scope** (slider) — 1–5 scale mapped to labels: Quick Fix | Small Task | Feature | Project | Epic. Default: 3 (Feature). Show label next to slider value.

### Constraints group (toggleable tag chips)

Display as a row of clickable pill-shaped chips. Clicking toggles on/off. Active chips are highlighted.

- **Performance sensitive** — default: off
- **Breaking change** — default: off
- **Requires migration** — default: off
- **Needs design review** — default: off
- **Third-party dependency** — default: off
- **Security implications** — default: off
- **Accessibility required** — default: off
- **Backwards compatible** — default: off

### Success Metrics group (toggleable tag chips)

Same chip UI as constraints. Active chips appear in the prompt.

- **Reduced load time** — default: off
- **Increased engagement** — default: off
- **Fewer support tickets** — default: off
- **Higher conversion** — default: off
- **Improved DX** — default: off
- **Better test coverage** — default: off
- **Reduced error rate** — default: off
- **User satisfaction** — default: off

### Notes group

- **Additional notes** (textarea, 4 rows) — free-form notes, edge cases, open questions. Default: empty. Placeholder: "Anything else to consider?"

## Preview

Render a **brainstorm card** — a styled document/card that presents the feature idea as a structured brief. The card should feel like a real planning document, not a raw data dump.

### Card sections

1. **Header** — Feature name in large display font. If empty, show "Untitled Feature" in dim text.
2. **Metadata bar** — Priority badge (color-coded), target user, scope label. Inline, compact.
3. **Problem** — Section with heading "Problem" and the problem statement text. If empty, show placeholder text in dim.
4. **Proposal** — Section with heading "Proposal" and the proposal text. Same empty handling.
5. **Constraints** — Only shown if at least one constraint is active. Display as small colored tags.
6. **Success Metrics** — Only shown if at least one metric is active. Display as small colored tags, different color from constraints.
7. **Notes** — Only shown if notes are non-empty. Displayed in a slightly different style (e.g., italic or muted background).

### Visual treatment

- Card has a subtle paper-like texture (CSS grain/noise pattern via background-image gradient overlay).
- Priority badge colors: Low = muted gray, Medium = amber, High = orange, Critical = red.
- Scope label uses an intensity treatment: larger scope = bolder/warmer color.
- Empty sections show ghost placeholder text so the card always has structure.

## Prompt rules

Generate a natural-language brainstorming brief. Only mention fields that have been filled in or toggled on.

Structure:

1. Start with the feature name as a title: "Feature: [name]" or "Brainstorm: [name]"
2. State the problem: "Problem: [statement]"
3. State the proposal: "Proposal: [statement]"
4. Mention target user and priority if non-default: "This is a [priority] priority feature targeting [target user]."
5. State scope using qualitative language: "Scope: This is a quick fix" / "This is a full project-scale effort"
6. List active constraints: "Constraints: Performance sensitive, requires migration"
7. List active metrics: "Success metrics: Reduced load time, improved DX"
8. Include notes if present: "Notes: [text]"
9. If everything is at defaults with no text fields filled, output: "Empty brainstorm — add a feature name and problem to get started."

Example prompt output:
> "Feature: Dark Mode Support
> Problem: Users report eye strain during evening use, and our app is one of the few competitors without dark mode.
> Proposal: Implement a system-preference-aware dark theme with manual toggle, using CSS custom properties for all color tokens.
> This is a high priority feature targeting end users. Scope: This is a feature-scale effort.
> Constraints: Needs design review, accessibility required.
> Success metrics: Increased engagement, user satisfaction.
> Notes: Consider auto-switching based on time of day. Check contrast ratios against WCAG AA."

## Presets

- **Blank Slate** — All fields empty, all chips off, all defaults: `{ featureName: '', problem: '', proposal: '', targetUser: 'End User', priority: 'Medium', scope: 3, constraints: all off, metrics: all off, notes: '' }`
- **Quick Fix** — Small, focused improvement: `{ priority: 'Medium', scope: 1, constraints: ['Backwards compatible'], metrics: ['Reduced error rate'] }`
- **User Pain Point** — Feature driven by user feedback: `{ priority: 'High', scope: 3, targetUser: 'End User', constraints: ['Needs design review', 'Accessibility required'], metrics: ['User satisfaction', 'Fewer support tickets'] }`
- **Tech Debt** — Internal improvement: `{ priority: 'Medium', scope: 3, targetUser: 'Developer', constraints: ['Performance sensitive', 'Backwards compatible'], metrics: ['Improved DX', 'Better test coverage'] }`
- **Big Bet** — Major strategic feature: `{ priority: 'Critical', scope: 5, targetUser: 'All Users', constraints: ['Needs design review', 'Security implications', 'Accessibility required'], metrics: ['Increased engagement', 'Higher conversion', 'User satisfaction'] }`

## Mistakes to avoid

- **Prompt is just a key-value dump** — Write it as a structured brief, not `priority: High, scope: 3`. Use natural language.
- **Card preview looks empty on first load** — Show ghost/placeholder text in all empty sections so the card has structure even before the user types anything.
- **Constraints and metrics chips don't feel clickable** — Use clear hover states, active indicator colors, and a subtle animation on toggle.
- **Textarea controls don't auto-resize** — Consider auto-growing textareas or at least ensure scroll works cleanly.
- **Feature name missing from prompt** — If the user typed a feature name, it must appear prominently in the prompt output.
- **Presets don't clear text fields** — When switching to "Blank Slate", all text fields must clear. When switching to other presets, only update the control values the preset defines — leave text fields as-is unless the preset explicitly sets them.
- **No visual priority differentiation** — The priority badge in the preview must use distinct, immediately readable colors. Don't rely on text alone.
