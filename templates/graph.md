# Code Map

Interactive codebase architecture diagram playground for exploring component relationships, data flow, and system layers. Users toggle layers, filter connections, annotate components, and copy a structured architecture description back into OpenCode.

## Controls

### View Presets (top bar, horizontal buttons)

- **Full System** — Show all layers and connections. Default active.
- **Data Flow** — Highlight data layer and all connections to/from it, dim others.
- **Client Only** — Show only client-side layers.
- **API Layer** — Show server + external layers, hide client internals.
- **Dependencies** — Highlight external services and their connections.

### Layers group (checkboxes)

- **Client** (checkbox) — default: checked. Color: `#60a5fa` (blue).
- **Server** (checkbox) — default: checked. Color: `#34d399` (green).
- **Data** (checkbox) — default: checked. Color: `#fbbf24` (amber).
- **External** (checkbox) — default: checked. Color: `#f87171` (red).
- **Infrastructure** (checkbox) — default: checked. Color: `#a78bfa` (purple).

### Connections group (checkboxes with color indicators)

- **HTTP/REST** (checkbox) — default: checked. Line style: solid. Color: `#60a5fa`.
- **WebSocket** (checkbox) — default: checked. Line style: dashed. Color: `#34d399`.
- **Database queries** (checkbox) — default: checked. Line style: dotted. Color: `#fbbf24`.
- **Event/message** (checkbox) — default: checked. Line style: dash-dot. Color: `#f87171`.

### Zoom controls

- **Zoom in** (+) button
- **Zoom out** (-) button
- **Reset zoom** button
- Current zoom level display (e.g., "100%")

## Preview

Render an **interactive SVG diagram** in the preview panel.

### Node rendering

- Each node is a rounded rectangle with:
  - Layer color as left border (4px)
  - Component name (bold, white text)
  - Brief description (small, gray text)
  - Background: `#1e293b` (slate-800)
- Nodes are positioned in horizontal swim-lanes per layer (Client at top, External at bottom)
- Nodes should be draggable within the SVG (mousedown/mousemove/mouseup)

### Connection rendering

- Lines between nodes using SVG `<line>` or `<path>` elements
- Line style and color based on connection type
- Arrow markers at the target end
- Opacity reduces to 0.15 when the connection type is unchecked

### Interaction

- **Click a node** to open a comment modal/popover:
  - Text input for annotation
  - "Save" and "Cancel" buttons
  - Saved comments show as a small badge/dot on the node
- **Comments sidebar** (right side, collapsible):
  - Lists all comments by component name
  - Each comment has a delete (x) button
  - Clicking a comment scrolls/highlights the node in the SVG

### Legend

- Show a legend at the bottom of the SVG with:
  - Layer colors and names
  - Connection line styles and names

### Layer visibility

- When a layer checkbox is unchecked:
  - Nodes in that layer fade to opacity 0.1
  - Connections to/from those nodes fade to opacity 0.1
- Do not remove elements from DOM — only adjust opacity for smooth transitions

## Generating the diagram content

When the user requests a code map playground, the agent must:

1. Analyze the user's codebase or description to identify components
2. Categorize each component into a layer (Client, Server, Data, External, Infrastructure)
3. Identify connections between components and their types
4. Generate the SVG with appropriate nodes and connections
5. If the codebase is too large, ask the user which subsystem to focus on

If no codebase context is available, generate a **sample architecture** (e.g., a typical web app with React frontend, Express API, PostgreSQL database, Redis cache, and an external payment service) so the playground is functional on first load.

## Prompt rules

Generate a structured architecture description. Only mention visible (checked) layers and annotated components.

Structure:

1. Start with "The system architecture consists of" or "Update the architecture:"
2. List visible layers and their components: "The client layer includes a React SPA and a service worker"
3. Highlight annotated components with user's comments: "The API Gateway (note: needs rate limiting) connects to..."
4. Describe visible connection patterns: "The client communicates with the server via REST and WebSocket"
5. If specific layers are hidden, note the focus: "Focusing on the client and server layers"
6. If all defaults with no comments, output: "Full system architecture view — no annotations or focus applied."

Example prompt output:
> "The system architecture focuses on the server and data layers. The Express API server connects to PostgreSQL via database queries and to Redis via a pub/sub event channel. The API Gateway (note: needs rate limiting) serves as the entry point. Update the API Gateway to include request throttling middleware."

## Presets

- **Full System** — All layers checked, all connection types visible: `{ layers: all checked, connections: all checked }`
- **Data Flow** — Data + Server layers highlighted, Client/External dimmed: `{ layers: { data: true, server: true, client: false, external: false, infra: false } }`
- **Client Only** — Only Client layer, HTTP connections: `{ layers: { client: true, server: false, data: false, external: false, infra: false }, connections: { http: true, ws: true, db: false, event: false } }`
- **API Layer** — Server + External layers: `{ layers: { client: false, server: true, data: false, external: true, infra: false } }`
- **Dependencies** — External + Infrastructure layers, all connection types: `{ layers: { client: false, server: false, data: false, external: true, infra: true } }`

## Mistakes to avoid

- **Static SVG with no interactivity** — Nodes must be clickable and draggable. Layers must toggle. This is not just an image.
- **Hardcoded layout that doesn't adapt** — Position nodes dynamically based on the number of components per layer. Don't overlap.
- **Comments lost on preset change** — User annotations must persist across preset changes and layer toggles.
- **No sample data** — If the agent has no codebase context, it must generate a realistic sample architecture. An empty diagram is useless.
- **Connections without labels** — Each connection line should show its type on hover or via the legend.
- **Ignoring layer colors** — Consistent color-coding across nodes, connections, legend, and checkboxes is essential for visual coherence.
