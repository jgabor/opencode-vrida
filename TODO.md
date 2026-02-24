# Vrida + OpenCode HTTP API Integration Plan

## Summary
Enable Vrida-generated single-file HTML rigs to use OpenCode as a backend over HTTP. The rig owns the interaction loop (UI, state, rendering), while OpenCode provides analysis/execution via API.

Pilot scope:
- Build one connected template first (`graph-live`) as the proving ground.
- Keep existing templates unchanged/offline-only for now.

## Product Decisions (Locked)
- Interaction happens in the rig UI, not in OpenCode TUI.
- OpenCode acts as backend API only.
- Pilot template: connected code map rig.
- LAN + localhost support without required auth (explicitly documented as high risk).
- Full-control behavior allowed when enabled by user.

## Deliverables
- Add `templates/graph-live.md` (connected variant template spec).
- Add `samples/graph-live.html` (working connected single-file rig).
- Update `SKILL.md` to define connected-rig exception to "no fetches":
  - Offline templates: no network calls.
  - Connected templates: allow local/LAN OpenCode HTTP calls, with graceful fallback.
- Update `README.md`:
  - Add "Connected rigs" setup.
  - Add CORS + hostname usage.
  - Add risk warning for unauthenticated LAN exposure.

## OpenCode API Surface (MVP)
- Connectivity:
  - `GET /global/health`
- Sessions:
  - `GET /session`
  - `POST /session`
  - `GET /session/status` (optional status indicator)
- Prompting:
  - `POST /session/{sessionID}/message`
  - body uses `parts: [{ type: "text", text: "..." }]`
- Repo introspection (optional in pilot, recommended):
  - `GET /file?path=<path>`
  - `GET /file/content?path=<path>`
  - `GET /find/symbol?query=<q>`
- Events + permissions (optional in MVP, strongly recommended):
  - `GET /global/event` (SSE)
  - `POST /permission/{requestID}/reply`
- Diff/status (optional power features):
  - `GET /file/status`
  - `GET /session/{sessionID}/diff`

## Rig Runtime Contract
- Still one HTML file, no external JS/CSS dependencies.
- Works disconnected with sample architecture + presets.
- Connected mode adds:
  - Base URL input (default `http://localhost:4096`)
  - Optional directory override (`x-opencode-directory` header)
  - Session selector + create session action
  - Generate graph action (LLM-backed via session message)
  - Optional stream/event panel + permission controls

## Graph Data Contract
Rig expects this JSON shape from model responses:

```json
{
  "meta": { "project": "string", "notes": "string" },
  "nodes": [
    { "id": "string", "name": "string", "layer": "client|server|data|external|infra", "description": "string" }
  ],
  "edges": [
    { "from": "nodeId", "to": "nodeId", "type": "http|ws|db|event", "label": "string" }
  ]
}
```

Hard safety caps in rig:
- `nodes.length <= 40`
- `edges.length <= 120`
- On overflow: truncate and show warning toast/banner.

## UX Tasks (Pilot: `graph-live`)
- Add "OpenCode Backend" control group:
  - Base URL field
  - Directory field (optional)
  - Session dropdown
  - "Connect" action (health check + version display)
  - "New session" action
- Add generation actions:
  - "Generate graph from repo"
  - Optional "Regenerate edges only"
  - Optional "Explain architecture"
- Add status feedback:
  - Connected/disconnected indicator
  - Last request status + errors
  - Loading state during requests

## Networking and CORS Notes
- Recommended serving mode:
  - Rig served over HTTP (`http://localhost:<rig-port>`)
  - OpenCode served on `http://localhost:4096`
- For LAN access:
  - OpenCode must run with `--hostname 0.0.0.0`
  - Allow rig origin with `--cors http://<lan-ip>:<rig-port>`
- Document that `file://` rig loading is not supported for connected mode due to CORS/origin behavior.

## Risk and Guardrails
- Because LAN/no-auth full control is permitted, include explicit warning in UI:
  - "Connected mode can run repository-affecting actions through OpenCode."
- Add an optional "danger mode" gate for write-capable actions (if enabled in pilot).
- Never silently auto-run destructive actions; all such actions are explicit button presses.

## Implementation Checklist
- [ ] Create `templates/graph-live.md`
- [ ] Build `samples/graph-live.html` with connected backend panel
- [ ] Implement health check and session management flows
- [ ] Implement graph-generation prompt + JSON extraction/parsing
- [ ] Enforce graph schema validation + caps + error handling
- [ ] Add optional SSE event panel (`/global/event`)
- [ ] Add optional permission response controls (`/permission/{requestID}/reply`)
- [ ] Update `SKILL.md` for connected template policy
- [ ] Update `README.md` with connected setup + CORS + safety notes
- [ ] Manual verify localhost flow end-to-end
- [ ] Manual verify LAN flow end-to-end

## Acceptance Criteria
- Connected rig can create/select a session and generate a graph from current repo.
- Existing offline templates behave exactly as before.
- Disconnected mode remains usable with presets/sample data.
- Errors from OpenCode API are surfaced clearly in rig UI.
- README and skill docs make setup and risks unambiguous.

## Deferred (Post-Pilot)
- Add connected variants for `canvas`, `review`, and `brainstorm`.
- Add richer event timeline and request replay.
- Add typed protocol helper shared across connected templates.
