# Document Critique

Document review playground for structured feedback on specs, RFCs, READMEs, proposals, or any prose document. Users review auto-generated suggestions, approve/reject each one, add comments, and copy the approved feedback as actionable instructions back into OpenCode.

## Controls

### Filter tabs (horizontal, top of sidebar)

- **All** — Show all suggestions. Display count badge. Default active.
- **Pending** — Show only suggestions not yet acted on. Display count badge.
- **Approved** — Show only approved suggestions. Display count badge.
- **Rejected** — Show only rejected suggestions. Display count badge.

### Suggestions sidebar (scrollable list)

Each suggestion is a card with:

- **Category badge** — One of: Clarity, Accuracy, Structure, Completeness, Style, Technical. Color-coded.
- **Title** — Short summary of the suggestion (e.g., "Clarify error handling behavior").
- **Description** — 1–2 sentence explanation of the suggestion.
- **Line reference** — Which section/lines of the document this applies to.
- **Action buttons:**
  - **Approve** (green) — Mark this suggestion as approved.
  - **Reject** (red) — Mark this suggestion as rejected.
  - **Comment** (blue) — Toggle a text input to add a comment.
- **Comment area** — When comment is toggled, show a textarea + "Save comment" button.
- **Status indicator** — Left border color: amber (pending), green (approved), red (rejected).

### Bulk actions (below filter tabs)

- **Approve All Pending** button
- **Reject All Pending** button
- **Reset All** button — return all suggestions to pending

## Preview

Render the **document content** in the main preview panel.

### Document rendering

- Display the document text with **line numbers** on the left (monospace, dim).
- Each line/paragraph has a **left border highlight** (4px):
  - Amber (`#f59e0b`) — has a pending suggestion
  - Green (`#22c55e`) — has an approved suggestion
  - Red (`#ef4444`) — has a rejected suggestion
  - No color — no suggestion for this section
- Multiple suggestions on the same section: use the "most active" color (pending > approved > rejected).

### Interaction

- **Click a suggestion card** in the sidebar → scroll the document to the referenced section and briefly highlight it (pulse animation).
- **Click a highlighted line** in the document → scroll the sidebar to the corresponding suggestion card and briefly highlight it.
- **Hover on a highlighted line** → show a tooltip with the suggestion title.

### Counters

Display at the top of the preview panel:

- Total suggestions: N
- Pending: N | Approved: N | Rejected: N
- Progress bar showing percentage completed (approved + rejected / total)

## Generating the suggestions

When the user requests a document review playground, the agent must:

1. Read the document provided by the user (file path, pasted text, or referenced content).
2. Generate 5–15 review suggestions across the categories: Clarity, Accuracy, Structure, Completeness, Style, Technical.
3. Each suggestion must reference a specific line range or section heading.
4. Suggestions should be actionable — not just "this could be better" but "rewrite this section to clarify the error handling behavior by specifying which errors are retried vs. fatal."

If no document is provided, generate a **sample document** (e.g., a short API specification or project README with intentional issues) with pre-populated suggestions so the playground is functional on first load.

## Prompt rules

Generate an actionable review summary. Only include **approved** suggestions and **user comments**. Rejected suggestions are excluded entirely.

Structure:

1. Start with "Apply the following review feedback:" or "Update the document with these changes:"
2. Group by category (Clarity, Structure, etc.)
3. For each approved suggestion:
   - Reference the section/line: "In the Error Handling section (lines 42–58):"
   - State the change: "Clarify which errors are retried vs. fatal"
   - Include user comment if present: "(Reviewer note: also add a retry count limit)"
4. If no suggestions are approved, output: "No changes approved — document looks good as-is."
5. If all suggestions are approved with no comments, output a streamlined list.

Example prompt output:
> "Apply the following review feedback:
>
> **Clarity:**
>
> - In the Error Handling section (lines 42–58): Clarify which errors are retried vs. fatal. (Reviewer note: also add a retry count limit of 3)
> - In the Introduction (lines 1–8): Add a one-sentence summary of the API's purpose.
>
> **Structure:**
>
> - Move the Authentication section before the Endpoints section for logical flow.
>
> **Completeness:**
>
> - Add rate limiting documentation — currently missing entirely."

## Presets

- **Fresh Review** — All suggestions set to pending: `{ statuses: all 'pending', filter: 'all' }`
- **Quick Scan** — Show only Clarity and Accuracy categories, hide others: `{ visibleCategories: ['Clarity', 'Accuracy'], filter: 'pending' }`
- **Deep Review** — Show all categories, expand all comment areas: `{ visibleCategories: all, commentsExpanded: true, filter: 'all' }`
- **Approve All** — Mark all suggestions as approved: `{ statuses: all 'approved', filter: 'approved' }`
- **Focus: Structure** — Show only Structure and Completeness categories: `{ visibleCategories: ['Structure', 'Completeness'], filter: 'pending' }`

## Mistakes to avoid

- **Suggestions without specific line references** — Every suggestion must point to a concrete section. "The document could be improved" is useless.
- **Including rejected suggestions in prompt** — Only approved items go in the prompt output. This is the entire point of the approve/reject workflow.
- **Losing comments on filter change** — User comments must persist when switching between filter tabs.
- **No visual linkage between sidebar and document** — Click-to-scroll in both directions is essential. Without it, users can't find where a suggestion applies.
- **Generic suggestion categories** — Use specific, meaningful categories. Don't lump everything under "General."
- **Too many suggestions** — 5–15 is the sweet spot. More than 20 becomes overwhelming. Prioritize high-impact suggestions.
- **No sample document** — If the user doesn't provide a document, generate a realistic one. An empty review playground is useless.
- **Approve/reject buttons don't update counters** — The progress bar and count badges must update immediately on every action.
