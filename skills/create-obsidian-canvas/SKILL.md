---
name: create-obsidian-canvas
description: Only invoked by user, do not trigger without explicit instructions. 
---
# Obsidian Canvas Creator 

Turn content into a `.canvas` file in the user's Obsidian vault by writing JSON directly through the **local-obsidian MCP server's `obsidian_put_content` tool**. No code execution, no shell — just construct the JSON in your head and send it as the `content` parameter.

Bias toward generating canvases that look hand-crafted, not generic: architecture diagrams use color-coded vendor groups with internal components, process flows go horizontally with phase markers and detail-cards below, methodology layouts are a phase row over a question-block grid, narrative canvases use spatial proximity instead of edges. Don't produce flat mind-map sludge.

**Bias toward space, not density.** The single most common LLM failure mode here is cramping everything onto a small grid. Canvases are infinite — use the space. See "Spacing — the #1 thing LLMs get wrong" below.

## Setup assumed

The local-obsidian MCP server is connected (the tool you need is `obsidian_put_content`, sometimes namespaced like `local-obsidian:obsidian_put_content`). The server points at the user's active vault — you only deal with vault-relative paths.

If the tool isn't available when the skill runs, stop and tell the user to enable the local-obsidian MCP server (Obsidian Settings → Community plugins → Local REST API + the MCP bridge of their choice).

## Workflow

1. **Understand the content.** What is it — a sequence (process), a system (architecture), a framework (methodology), branching options (decision tree), a brain dump (mind map), or buckets (kanban)? Don't ask if it's obvious — pick and proceed.
2. **Pick a layout pattern** from the chooser below.
3. **Choose a filename and folder.** Default folder: `Canvases/` in vault root. If the content is clearly TTRPG / campaign / story material, default to `TTRPG/` instead. Filename: title-cased, spaces allowed (Obsidian handles them). **Always include the `.canvas` extension** in the path.
4. **Construct the JSON** following the spec in this file. Generate 16-char lowercase hex IDs by hand (see "ID generation" below). Mentally validate against the checklist at the end of this file.
5. **Write it** by calling `obsidian_put_content` with `filepath` set to the vault-relative path and `content` set to the JSON string. That's it — Obsidian picks up the new file automatically; the user can click it open from the file explorer.

## Step 1 — Pick the layout

| Content shape | Layout |
|---|---|
| Sequential steps, phases, pipeline (1 → 2 → 3) | **horizontal_flow** |
| Systems, services, vendors with internal components | **architecture** |
| Workshop / framework / methodology: phase row on top, detail blocks below | **phase_grid** |
| Central concept with related items radiating out | **hub_and_spoke** |
| Categories with cards inside (todo/doing/done, buckets) | **kanban** |
| Tree of decisions, narrative branches, story scenes | **branching_tree** |

Patterns compose. An architecture diagram can have a small horizontal_flow inside one of its groups. A phase_grid can have hub_and_spoke clusters under each phase. Full details in the "Layout patterns" section below.

## Step 2 — Construct the JSON

### Top-level shape

```json
{
  "nodes": [ ... ],
  "edges": [ ... ],
  "metadata": { "version": "1.0-1.0", "frontmatter": {} }
}
```

`metadata` is optional but Obsidian writes it on every save — include it for clean round-trips.

### ID generation (manual, no code)

Every node and edge needs a unique 16-character lowercase hex ID. Without code execution, generate them by hand: pick characters from `0-9a-f`, exactly 16 chars, e.g. `6f0ad84f44ce9c17`, `a1b2c3d4e5f67890`, `7e3f1d2c8b4a9f06`. **Don't reuse IDs.** Keep a mental list as you build the canvas.

A reliable pattern: use the first 8 chars to encode position (e.g. `n0000001` for node 1) and the last 8 random — but make sure every char is in `[0-9a-f]`. Or just write random-looking hex; what matters is uniqueness.

### Node types

#### Text node

```json
{
  "id": "6f0ad84f44ce9c17",
  "type": "text",
  "x": 0, "y": 0, "width": 400, "height": 200,
  "text": "# Hello\n\nThis is **Markdown** with a [[wikilink]].",
  "color": "4"
}
```

`text` is Obsidian-flavored Markdown: headers (`#`, `##`, `###`), bold/italic, bullets (`- item`), checkboxes (`- [ ]`), wikilinks (`[[Note]]`), embeds (`![[Note]]`, `![[image.png]]`), code fences (including ` ```mermaid `), raw HTML.

**Newline pitfall**: in JSON strings, newlines must be `\n` (two characters: backslash + n), not literal newlines. If you embed a multi-line Markdown block, escape every newline.

#### File node

```json
{
  "id": "a1b2c3d4e5f67890",
  "type": "file",
  "x": 500, "y": 0, "width": 400, "height": 300,
  "file": "Knowledge/Attachments/Logo.png",
  "subpath": "#Section heading"
}
```

`file` is vault-relative. `subpath` is optional — starts with `#` for headings, `#^` for block references. Add `"portal": true` to embed another `.canvas` interactively.

#### Link node

```json
{
  "id": "c3d4e5f678901234",
  "type": "link",
  "x": 1000, "y": 0, "width": 400, "height": 200,
  "url": "https://obsidian.md"
}
```

#### Group node

```json
{
  "id": "d4e5f6789012345a",
  "type": "group",
  "x": -50, "y": -50, "width": 1000, "height": 600,
  "label": "Project Overview",
  "color": "4"
}
```

A visual container. Children are nodes whose `(x, y, width, height)` fall inside the group's bounds — there's no parent pointer. Always give groups a `label`, usually a `color`.

#### styleAttributes (Obsidian extension)

Optional on nodes; the user relies on these:

```json
"styleAttributes": { "textAlign": "center", "border": "invisible" }
```

- `textAlign`: `"left"` (default), `"center"`, `"right"`. Use `"center"` on short label nodes.
- `border`: `"invisible"` hides the node frame — useful on image-embedding file nodes.

Omit `styleAttributes` entirely when not needed.

### Edges

```json
{
  "id": "0123456789abcdef",
  "fromNode": "6f0ad84f44ce9c17",
  "fromSide": "right",
  "toNode": "a1b2c3d4e5f67890",
  "toSide": "left",
  "label": "leads to"
}
```

Fields:

| Field | Required | Default | Notes |
|---|---|---|---|
| `id` | yes | — | Unique across all nodes and edges. |
| `fromNode` / `toNode` | yes | — | Must reference existing node IDs. |
| `fromSide` / `toSide` | no | floating | `top` / `right` / `bottom` / `left`. |
| `fromEnd` | no | `"none"` | `"none"` or `"arrow"`. |
| `toEnd` | no | `"arrow"` | `"none"` or `"arrow"`. |
| `color` | no | — | Preset `"1"`–`"6"` or hex. |
| `label` | no | — | Text on the edge. |

Explicit `fromSide`/`toSide` makes edges stable when nodes move. Leave them off only for floating / non-directional relationships.

Edge style extensions Obsidian honors:

```json
{
  "fromFloating": false,
  "toFloating": false,
  "styleAttributes": {
    "pathfindingMethod": "square",
    "path": "short-dashed"
  }
}
```

- `pathfindingMethod: "square"` — rectilinear (L-shaped) routing. Prefer this for architecture diagrams.
- `path: "short-dashed"` — dashed line, for "reference" / non-flow edges.

### Color semantics

Match the user's conventions — don't pick arbitrarily:

| Preset | Conceptual color | Meaning in user's canvases |
|---|---|---|
| `"1"` | red | Pain points, entry points, human-driven steps, problems |
| `"2"` | orange | External triggers, integrations, side-channels |
| `"3"` | yellow | Done / status / categorization / decision branches |
| `"4"` | green | Stable / production / positive / completed |
| `"5"` | cyan | Infrastructure / cloud / state / process containers |
| `"6"` | purple | Orchestrators / brains / AI agents / coordinators |

Hex colors (`"#163b6e"`, `"#ff382e"`) are fine for one-off accents. Child nodes inside a colored group usually inherit the group's color number.

### Sizing conventions

These come from the user's existing canvases — match them and the output will look native. **Treat these ranges as MINIMUMS for breathing room, not maximums.** Err larger.

- **Text node**: 280–320 wide × 60–100 tall for headers; 380–450 × 200–400 for content cards.
- **Group nodes**: 50–80 px padding around child nodes on every side; don't hug too tight.
- **Sibling spacing in a flow**: see the Spacing section below — 80–120 px is too tight for most layouts.
- **Grid alignment**: snap positions to multiples of 20 for clean rendering.
- **Negative coordinates** are valid — the canvas is infinite, `(0, 0)` is wherever you want it.

### Spacing — the #1 thing LLMs get wrong

The default LLM failure is cramping. A correctly-sized canvas feels almost too sparse on paper but reads cleanly in Obsidian. **Calibrate against these numbers from real hand-built canvases:**

| Dimension | Wrong (cramped) | Right (breathing) |
|---|---|---|
| **Horizontal stride** (card x to next card x) for a card with width 300 | 340 px (40 px gap) | **480–500 px (180–200 px gap)** |
| **Vertical stride** (card y to next card y) for a card with height ~110 | 160 px (50 px gap) | **300–340 px (190–230 px gap)** |
| **Gap between sibling groups** | 60 px | **150–400 px** depending on canvas size |
| **Group padding around children** | 20 px (tight) | **40–60 px on every side** |
| **Title / legend card** | 400×160 | **560×400** — give explanatory cards real estate |
| **Group containing 4–6 cards in a 2×N grid** | 700×460 | **1100×700 to 1500×1000** |
| **Group containing 8 cards in a 3×3 grid** | 1100×800 | **~2450×1000** |
| **Total canvas span (8–25 nodes)** | 1500×1500 | **3500–4500 wide, 2500–3500 tall** is normal |

**Rules of thumb when laying out:**

1. **Aim for "gap ≈ card width" between cards in a row.** A 300 px-wide card wants a 480–500 px x-stride. The visual gap should feel about as wide as the card itself.
2. **Aim for "gap ≈ 2× card height" between rows.** A 110 px-tall card wants ~300 px y-stride.
3. **Groups get their own region.** Don't butt two groups against each other. Leave 150–400 px of empty canvas between group bounding boxes.
4. **Title/header/legend cards are big.** They contain real prose; size them at 500–600 wide × 300–400 tall, not 400×160.
5. **When in doubt, double the spacing.** If your layout fits in a 1500×1500 box, multiply both dimensions by ~2. The canvas will still read densely on a normal monitor.
6. **Don't optimize for fitting on one screen.** Obsidian's canvas zoom and pan are excellent. A canvas that requires panning is normal and good.

**Calibration example (PRD-covered cluster, 8 cards in roughly a 3×3 grid):**
- Card size: `300×145` (not the minimum 300×110)
- x positions: `−440, −100, 240` → stride **340… wait, that's tight**. Use stride **490** instead: `0, 490, 980`.
- y positions: `100, 425, 750` → stride **325**.
- Enclosing group: ~`width = 3×490 + 2×40 padding = 1550`, `height = 3×325 + 2×40 = 1055`.

If your numbers come out smaller than this for a similar card count, increase them before writing the file.

## Step 3 — Write to the vault

Call the MCP tool. Typical signature:

```
obsidian_put_content(
  filepath="Canvases/My New Diagram.canvas",
  content="<the full JSON as a string>"
)
```

The exact tool name may be namespaced (e.g. `local-obsidian:obsidian_put_content`) depending on how the MCP server is registered. The two parameters are always `filepath` (vault-relative, **must include the `.canvas` extension**) and `content` (the JSON serialized as a string).

`obsidian_put_content` **overwrites** if the file already exists — confirm with the user before replacing an existing canvas.

After the call succeeds, tell the user where you wrote it (e.g. "Written to `Canvases/My New Diagram.canvas` — open it from the file explorer"). Don't promise that it auto-opens; the MCP tool just writes the file.

---

## Layout patterns

### 1. horizontal_flow

**Shape**: left-to-right sequence of step cards, each connected `fromSide: right → toSide: left`. Optionally, detail-cards hang below each step. Optionally, lane groups span the full width.

**Pick when**: sequential — phases, steps, pipelines, workflows. Words like "first / then / finally", numbered lists, arrows.

**Coordinates**:

| Element | Size | Spacing |
|---|---|---|
| Step card | 280×120 to 320×180 | **480–500 px horizontal stride** (not 350) |
| Detail card below | 280×400 to 320×900 | **120–160 px below step card** (not 40) |
| Lane group | full width × 165–375 tall | spans all steps; 40–60 px padding |

Step cards: `color "1"` for human-driven, `"4"` or `"6"` for AI/automation. Entry/exit phase markers: `"2"` or `"5"`.

**Layout**:
```
[Step 1] → [Step 2] → [Step 3] → [Step 4]      ← y=320 constant, x stride ~490
   │          │          │          │
   ▼          ▼          ▼          ▼          ← toEnd:"none" for clean vertical drops
[detail]   [detail]   [detail]   [detail]      ← y=600 (≥160 below step bottom)
```

Step positions: `x = 0, 490, 980, 1470, ...`, `y = 320`. Detail cards at `y = 600`.

Edges: horizontal use `fromSide: "right"`, `toSide: "left"`, default `toEnd: "arrow"`. Vertical step→detail use `fromSide: "bottom"`, `toSide: "top"`, `toEnd: "none"`.

### 2. architecture

**Shape**: color-coded groups represent system boundaries (Azure, n8n, vendors), with text/file nodes inside as components. Edges cross group boundaries to show data flow, using `pathfindingMethod: "square"`.

**Pick when**: systems, services, vendors, integrations, software architecture. Words like "API", "MCP server", "database", "microservice".

**Coordinates**:

| Element | Size |
|---|---|
| System/vendor group | 400×300 min, up to 1500×800 for big platforms |
| Component card | 250×100 to 350×200 |
| Component card with body | 350×170 to 400×300 |
| Padding inside group | **40–60 px** (not 20) |
| Gap between groups | **150–300 px** (not 60) |

Color mapping:

| Group represents | Color |
|---|---|
| Cloud platform (Azure, AWS, GCP) | `"5"` cyan |
| AI orchestrator (Copilot Studio, n8n) | `"6"` purple |
| External vendor (Recall.ai, Gladia) | `"2"` orange |
| Internal stable system / project | `"4"` green |

Children inherit the group's color number — visual grouping is reinforced.

**Layout**:
```
┌─ Scoro (green) ─────────┐         ┌─ Azure (cyan) ─────────────┐
│ [MCP Server]  [API]     │         │ [Jobs] [MCP] [Search]      │
└─────────────────────────┘         └────────────────────────────┘
        │                                    │
        ▼                                    ▼
   [Copilot Studio (purple)]    →     [n8n (purple)]
                                             │
                                             ▼
                                        [Recall.ai] [Gladia]  (orange)
```

Edges: explicit `fromSide`/`toSide` based on geometry. Add `styleAttributes.pathfindingMethod: "square"` for L-shaped routing. Label edges with the data being passed.

### 3. phase_grid

**Shape**: a top row of title/context/evergreen panels, a phase-marker row (color `"1"` red), then under each marker a tall detail card. Comb structure.

**Pick when**: workshops, methodologies, frameworks where each phase has the same kind of detail (questions, deliverables).

**Coordinates**:

| Element | Size | Position |
|---|---|---|
| Title / context panels | 400×370, 400×370, 611×370 | y = 0, x = 0, 440, 880 |
| Phase markers | 280×120 | y = 480, x stride 320 |
| Detail cards | 280×730 | y = 640, x stride 320 (aligned with markers) |

8 phases is typical (e.g. SDLC: idea / requirements / design / coding / review / testing / deploy / monitor). Adjust count to content; keep x stride constant.

(Phase_grid is the one pattern where a tighter x stride is correct — the comb structure depends on visual alignment between markers and details.)

**Layout**:
```
[Title]     [Tips]      [Evergreens]
─────────────────────────────────────────────────────────
[Idea] [Reqs] [Design] [Coding] [Review] [Test] [Deploy] [Monitor]   ← color "1"
  │      │       │        │        │       │       │        │
  ▼      ▼       ▼        ▼        ▼       ▼       ▼        ▼
[5 Qs] [5 Qs]  [5 Qs]   [5 Qs]   [5 Qs] [5 Qs]  [5 Qs]   [5 Qs]    ← 280×730 each
```

Edges: phase row uses `fromSide: "right"`, `toSide: "left"`. Phase → detail uses `fromSide: "bottom"`, `toSide: "top"`, `toEnd: "none"`.

### 4. hub_and_spoke

**Shape**: central concept node with several related nodes around it, each connected by an edge. Satellites may have their own small children.

**Pick when**: single concept with branching attributes / inputs / outputs.

**Coordinates**:

| Element | Size | Position |
|---|---|---|
| Central node | 250×60 to 300×100 | center of canvas (use negative coords) |
| Satellite nodes | 250×60 | **350–500 px radial from center** (not 200) |
| Satellite children | 250×60 | stacked below satellites, 120 px spacing |
| Optional groups around clusters | bound + 40 px padding | — |

For 4–6 satellites, use compass positions (N/S/E/W or NE/NW/SE/SW). For more, clock positions.

Edges: hub → satellite uses `fromSide` based on the hub side facing the satellite. Satellite → its children: vertical (`fromSide: "bottom"`, `toSide: "top"`).

### 5. kanban

**Shape**: 3+ vertical groups side by side, each labeled with a category (Todo / Doing / Done, or any bucket scheme). Cards sit inside groups. Usually no edges — categorization is positional.

**Pick when**: clearly categorical content with N buckets. Also good for game state breakdowns / trading-card layouts.

**Coordinates**:

| Element | Size |
|---|---|
| Column group | 320×600 to 360×800 |
| Card in column | 260×80 to 280×120 |
| Gap between columns | **100–150 px** (not 50) |
| Gap between cards in a column | 20–40 px |

Each column gets a different color (`"1"` / `"3"` / `"4"` is the classic Todo/Doing/Done).

Edges: usually none. If showing dependencies across columns, thin uncolored edges with no labels.

**Composition note**: kanban-style grouping pairs nicely with free-floating "rule cards" outside the groups that visually connect to specific keywords. Use this when categories also have associated longer-form content.

### 6. branching_tree

**Shape**: a root node at top, branches splitting downward. Or a narrative graph where positions reflect story geography (rooms, locations) rather than strict hierarchy.

**Pick when**: decision trees, story plotting, hierarchical breakdowns, knowledge graphs, RPG campaign maps.

**Coordinates** (strict tree):

| Element | Size | Spacing |
|---|---|---|
| Root | 250–400 wide | (0, 0) |
| Level 1 children | 250×60 to 300×100 | **300–400 px below root**, fanned horizontally |
| Level 2 children | same | 300 px below parent |
| Sibling spacing | **200 px between adjacent** | grows with subtree width |

**Spatial graph (RPG-style)**:
- Cluster locations on the left, characters on the right, timeline along the top.
- Groups with `border: "invisible"` label regions without a visible boundary.
- Distances reflect narrative proximity, not strict hierarchy.

Edges: tree uses `fromSide: "bottom"`, `toSide: "top"`. Spatial uses floating edges (no explicit sides).

---

## Composition patterns

A few patterns combine the above:

- **Architecture + horizontal_flow inside a group**: an `n8n` group contains a left-to-right pipeline of action nodes.
- **Phase_grid + hub_and_spoke detail**: each phase's detail card is a small hub with sub-questions.
- **Kanban with rule overlays**: categorical groups + free-floating detail cards next to specific keywords.
- **Branching_tree with embedded content**: tree nodes are `file` embeds (`![[Character#Heading]]`) pulling live content from elsewhere in the vault.

## Defaults to avoid generic output

The fastest way to make a canvas look "AI-generated" is to put everything on a flat grid with no color and no groups, packed tight. The fastest way to avoid that:

1. **Use groups even for small canvases** (3+ nodes). Group by semantic category.
2. **Use colors meaningfully** (see color mapping). Inherit group color in children.
3. **Use Markdown headers and bullets** in text node content, not flat sentences.
4. **Use spatial proximity** to imply relationship before resorting to edges.
5. **Set `fromSide`/`toSide` explicitly** on edges with a clear directional relationship.
6. **Spread out.** When in doubt about spacing, multiply by 1.5–2× before writing. See "Spacing — the #1 thing LLMs get wrong".

---

## Mental validation checklist

Before calling `obsidian_put_content`, walk through this list:

1. **Unique IDs**: every `id` across nodes + edges combined is unique. Re-read all IDs and check.
2. **Edge references**: every edge's `fromNode` and `toNode` matches an existing node `id` exactly.
3. **Required fields per type**:
   - `text` → has `text`
   - `file` → has `file`
   - `link` → has `url`
   - `group` → typically has `label`
   - All nodes → `id`, `type`, `x`, `y`, `width`, `height`
4. **Type values** are exactly one of `"text"`, `"file"`, `"link"`, `"group"`.
5. **Side values** (when present) are exactly one of `"top"`, `"right"`, `"bottom"`, `"left"`.
6. **JSON validity**: balanced braces/brackets, commas between array items and object fields, no trailing commas, no smart quotes (`"` not `"` `"`).
7. **String escaping**: newlines inside `text` fields are `\n`, double-quotes inside strings are `\"`, backslashes are `\\`.
8. **Filename**: ends in `.canvas`.
9. **Spacing sanity check**: for each row of sibling cards, `x_stride ≥ card_width + ~180`. For each column, `y_stride ≥ card_height + ~190`. Group bounds extend at least 40 px past child bounds on every side. Total canvas width for 8+ nodes is at least ~2500 px. If any of these fail, expand before writing.

If anything fails, fix and re-check before sending.

---

## Worked example

A small horizontal_flow with two steps and one detail card, three IDs total (two nodes + one edge):

```json
{
  "nodes": [
    {
      "id": "a1b2c3d4e5f60001",
      "type": "text",
      "x": 0, "y": 320, "width": 280, "height": 120,
      "text": "## Discover\n\nUser interviews & research.",
      "color": "1"
    },
    {
      "id": "a1b2c3d4e5f60002",
      "type": "text",
      "x": 490, "y": 320, "width": 280, "height": 120,
      "text": "## Define\n\nSynthesize problem statement.",
      "color": "1"
    }
  ],
  "edges": [
    {
      "id": "a1b2c3d4e5f60101",
      "fromNode": "a1b2c3d4e5f60001",
      "fromSide": "right",
      "toNode": "a1b2c3d4e5f60002",
      "toSide": "left"
    }
  ],
  "metadata": { "version": "1.0-1.0", "frontmatter": {} }
}
```

This is what gets passed as the `content` parameter (as a string). When sending through the MCP tool, the whole thing is one string — make sure inner double-quotes are escaped if your tool layer doesn't accept raw multi-line strings.

---

## Edge cases worth knowing

- **Tool name variations**: the tool may appear as `obsidian_put_content`, `local-obsidian:obsidian_put_content`, or `mcp__<uuid>__obsidian_put_content` depending on how the MCP bridge registers it. Use whichever name your environment exposes.
- **Vault path is implicit**: the MCP server is already pointed at one vault. You pass vault-relative paths, never absolute.
- **Overwriting**: `obsidian_put_content` overwrites existing files. Confirm with the user before replacing an existing canvas.
- **Large content**: very large canvases (thousands of nodes) can hit MCP message-size limits. For canvases this big, build in chunks and append — but this is rare; most canvases stay well under 100 KB.
- **Subfolders**: if the folder doesn't exist (`Canvases/`, `TTRPG/`), most local-obsidian MCP servers create it automatically. If the put fails with a "folder not found" error, ask the user to create the folder once.
- **Embedded canvases**: `type: "file"` nodes with `"portal": true` embed another canvas inline. Reference path is vault-relative.
- **Multiple vaults**: most local-obsidian MCP servers are bound to a single vault. If the user has multiple vaults, you can only write to whichever one is configured.

## When NOT to use this skill

- The user wants a Markdown note with content, not a spatial layout — use a regular `.md` file (you can write it with `obsidian_put_content` too, just with a `.md` extension and Markdown content).
- The user wants a Mermaid diagram inside a single note — write the Mermaid block into a `.md`, no canvas needed.
- The user wants a presentation — that's `pptx_generator` territory, not canvas.
