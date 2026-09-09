# Project Pulse Implementation Plan

## Summary

Mona's team needs a lightweight, static **Project Pulse** dashboard that helps contributors quickly see which projects are active, who owns them, their current status, recent activity, and priority/risk level. The final deliverable is a small static app served from `app/` with a polished card-based UI, plus a VS Code launch configuration that runs the dashboard from a local HTTP server and opens `index.html` (never a directory listing).

The AI agent team from `.github/agents/` will build it under the standard flow documented in `docs/agent-team.md`:

- **Orchestrator** coordinates work, assigns file scopes, and enforces sequencing.
- **Planner** (this document) defines phases, dependencies, and file ownership.
- **Designer** owns visual design, layout, information hierarchy, accessibility, and CSS.
- **Coder** owns HTML markup, JSON data, and the `.vscode/launch.json` runnable-app support file.

The build is intentionally deterministic to satisfy the validation in `.github/workflows/3-step.yml` and `scripts/validate-exercise.sh`: exact CSS hooks (`.dashboard`, `.project-card`), styling phrases (`border-radius`, `box-shadow`), markup phrases (`project-card`, `status`, `recentActivity`, `priority`), a top-level `projects` array with the required fields, and a launch config named exactly **Run Project Pulse Dashboard** targeting `index.html`.

## Ordered Implementation Steps

### Phase 1 — Data contract (Coder)
Define the shape and seed content of `app/project-data.json`. The data contract is the foundation for both the markup (Coder) and the visual treatment (Designer, e.g., status badge variants). Nothing else can be finalized without this shape.

**Deliverable:** `app/project-data.json` with a top-level `projects` array. Each project must include:
- `name` (string)
- `owner` (string)
- `status` (string — e.g., `"On track"`, `"At risk"`, `"Blocked"`, `"Shipped"`)
- `recentActivity` (string — short human-readable sentence)
- `priority` (string — e.g., `"High"`, `"Medium"`, `"Low"`)

Include 3–6 sample projects covering multiple status and priority values so the Designer can style all badge variants.

### Phase 2 — Parallel build of markup + styles

Because the data contract is now fixed and the two files touch non-overlapping scopes, the Orchestrator runs these tasks **in parallel**:

#### Phase 2a — Markup and data wiring (Coder)
Build `app/index.html`:
- Exact title text **Project Pulse** in a visible heading (also fine as `<title>`).
- `<link rel="stylesheet" href="styles.css">`.
- Reference `project-data.json` (either via `fetch('project-data.json')` in a `<script>` block, or via a comment/data-attribute plus JS load — the phrase `project-data.json` must appear in the file).
- A container element with `class="dashboard"`.
- Rendered project cards each with `class="project-card"` (rendered via inline JS reading `project-data.json`).
- Each card must display the project's `status`, `recentActivity`, and `priority` values (the literal words `status`, `recentActivity`, and `priority` must appear in the source, which is naturally satisfied by field references / labels).
- Semantic, accessible markup: `<main>`, `<header>`, headings in order (`h1` → `h2`), `role`/`aria-label` on status badges, `alt`/`aria-*` where needed.

#### Phase 2b — Visual design and styling (Designer)
Build `app/styles.css`:
- `.dashboard` selector for the grid/flex layout.
- `.project-card` selector for the card treatment.
- Required phrases `border-radius` and `box-shadow` present in the card styles.
- Status badge classes (e.g., `.status--on-track`, `.status--at-risk`, `.status--blocked`) with accessible color contrast.
- Priority visual treatment (e.g., left border stripe or pill).
- Responsive grid: single column on narrow viewports, multi-column on wider viewports (`@media` queries or `grid-template-columns: repeat(auto-fill, minmax(...))`).
- Readable typography, generous spacing, focus-visible outlines.

### Phase 3 — Launch configuration (Coder)
After the app files exist, create `.vscode/launch.json` as **strict JSON with no comments**:
- One configuration named exactly **Run Project Pulse Dashboard**.
- Runs `python3 -m http.server 5500`.
- `cwd`: `${workspaceFolder}/app` (so the server root is `app/`).
- `serverReadyAction` that opens `http://localhost:%s/index.html` in the external browser.
- Type/request appropriate for a Node/debugpy launch that just spawns the command (a `type: "node"` config using `runtimeExecutable: "python3"` with `runtimeArgs: ["-m", "http.server", "5500"]`, or equivalent). The exact runtime is not asserted by CI; the asserted phrases are the name, `index.html`, and JSON validity.

### Phase 4 — Integration verification (Orchestrator + learner)
Local sanity check before the learner commits (see **Validation expectations**).

## File Assignments

| File | Responsible agent | Notes |
|---|---|---|
| `app/project-data.json` | **Coder** | Top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`. Must parse via `python3 -m json.tool`. |
| `app/index.html` | **Coder** | Title "Project Pulse"; references `styles.css` and `project-data.json`; renders `.project-card` markup with `status`, `recentActivity`, `priority`; container has class `dashboard`. |
| `app/styles.css` | **Designer** | Includes `.dashboard`, `.project-card`, `border-radius`, `box-shadow`; polished, responsive, accessible. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments; configuration named **Run Project Pulse Dashboard**; `cwd` = `${workspaceFolder}/app`; opens `index.html` (not a directory listing). |

## Designer Responsibilities

- **Layout:** Header with "Project Pulse" title and short subtitle; main `.dashboard` region as a responsive card grid.
- **Information hierarchy:** Project `name` is the most prominent element; `owner` is secondary; `status` is a visible badge; `recentActivity` reads as a short sentence; `priority` is signaled visually (color + label, not color alone).
- **Accessibility:**
  - Sufficient contrast (WCAG AA) for text and badges.
  - Priority/status conveyed by shape or label as well as color (colorblind-safe).
  - Keyboard focus styles (`:focus-visible`).
  - Semantic landmark structure and heading order.
- **Visual polish:** Rounded corners (`border-radius`), soft elevation (`box-shadow`), consistent spacing scale, readable font stack.
- **Required CSS hooks:** `.dashboard` and `.project-card` selectors present in `app/styles.css`.
- **Status badges:** Distinct treatments for the sample statuses used in `project-data.json`.
- **Responsive behavior:** Single-column layout below ~600px; 2 columns for mid widths; 3+ columns on wide screens. Cards should never overflow or truncate essential fields.

## Coder Responsibilities

- **`app/project-data.json`:** Valid JSON with a top-level `"projects"` array. Each object contains `name`, `owner`, `status`, `recentActivity`, `priority`. Use values that exercise multiple badge variants (e.g., one `"At risk"`, one `"Blocked"`, one `"On track"`, one `"Shipped"`).
- **`app/index.html`:**
  - Exact string "Project Pulse" visible in the page.
  - `<link rel="stylesheet" href="styles.css">`.
  - Loads `project-data.json` at runtime (e.g., `fetch('project-data.json')`) and renders one `.project-card` per project into the `.dashboard` container.
  - Each card visibly labels and prints the `status`, `recentActivity`, and `priority` values.
  - Handles empty state and missing fields gracefully (see **Edge cases**).
- **`.vscode/launch.json`:**
  - Strict JSON, no `//` or `/* */` comments (must parse via `python3 -m json.tool`).
  - Exactly one configuration named **Run Project Pulse Dashboard**.
  - Serves the `app/` directory (`cwd`: `${workspaceFolder}/app`) via `python3 -m http.server 5500`.
  - `serverReadyAction` opens `http://localhost:%s/index.html` externally.
  - Must open the dashboard UI, never a directory listing.

## Dependencies Between Steps

1. **Phase 1 (data shape) → Phase 2a (markup)**: Coder can't render fields it hasn't defined.
2. **Phase 1 (data shape) → Phase 2b (styles)**: Designer needs the set of `status`/`priority` values to design badge variants.
3. **Phase 2a and 2b → Phase 3 (launch config)**: The app must exist before the launch config points at it (also lets the learner smoke-test locally).
4. **Phase 3 → Phase 4 (verification)**: Verification requires all four files.

## Parallel vs. Sequential Work

**Parallel (Phase 2a and Phase 2b):**
- Coder edits `app/index.html`; Designer edits `app/styles.css`. Non-overlapping file scopes.
- Both only *read* from the already-finalized `app/project-data.json`.
- They coordinate on **class name contracts only** (`.dashboard`, `.project-card`, status/priority modifier classes) — these are fixed in this plan, so no runtime coordination is needed.

**Sequential:**
- Phase 1 must precede Phase 2 (data contract is upstream of both markup and styling decisions).
- Phase 3 must follow Phase 2 (launch config validates a real app on disk).
- Phase 4 must be last.

**Why not parallelize the launch config with Phase 2?** It's a small file and Coder owns both `index.html` and `launch.json`; overlapping agent load risks path/name drift. Sequencing keeps the launch config aligned with whatever markup Coder actually shipped.

## Edge Cases to Handle

- **Missing fields in a project object:** The render loop should tolerate absent `owner`, `recentActivity`, or `priority` and render a graceful placeholder (e.g., `—`) rather than `undefined`.
- **Empty `projects` array:** Show a friendly empty-state message ("No projects yet") instead of a blank dashboard.
- **Many projects:** Grid should wrap; no horizontal scroll; cards keep consistent height or gracefully vary.
- **Long text:** `recentActivity` and `name` should wrap; use `overflow-wrap: anywhere` or `word-break` to avoid layout blowout. Avoid truncation that hides critical info.
- **Unknown `status` or `priority` values:** Fall back to a neutral badge style rather than an unstyled element.
- **Accessibility contrast:** Verify badge color/text pairs meet WCAG AA (~4.5:1). Do not signal state with color alone.
- **Responsive breakpoints:** Test ~360px, ~768px, ~1200px widths. Cards should never be narrower than their padding + content minimum.
- **Fetch from `file://`:** If the learner opens `index.html` directly instead of via the launch config, `fetch('project-data.json')` may fail due to CORS. Document that the dashboard must be viewed via **Run Project Pulse Dashboard**. Optionally, log a helpful console message on fetch failure.
- **JSON strictness for `launch.json`:** No trailing commas, no comments — VS Code tolerates JSONC but `python3 -m json.tool` (used in CI) does not.

## Validation Expectations

Before the learner asks Copilot CLI to commit, verify locally:

1. **Files exist:**
   - `app/index.html`
   - `app/styles.css`
   - `app/project-data.json`
   - `.vscode/launch.json`
2. **JSON parses:**
   - `python3 -m json.tool app/project-data.json > /dev/null`
   - `python3 -m json.tool .vscode/launch.json > /dev/null`
3. **Required phrases (mirrors `.github/workflows/3-step.yml`):**
   - `app/index.html` contains: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
   - `app/styles.css` contains: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
   - `app/project-data.json` contains keys: `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`.
   - `.vscode/launch.json` contains: `Run Project Pulse Dashboard`, `index.html`.
4. **Runs correctly:**
   - Open **Run and Debug**, pick **Run Project Pulse Dashboard**, press play.
   - Browser opens `http://localhost:5500/index.html` (not a directory listing).
   - Cards render with visible name, owner, status, recent activity, and priority.
   - Stop the server before committing.
5. **CI:** `.github/workflows/3-step.yml` will re-run all of the above key-phrase and JSON checks on push to `main` when `app/**` or `.vscode/launch.json` changes.

## Open Questions

1. **Status vocabulary:** Should status values be fixed (`On track`, `At risk`, `Blocked`, `Shipped`) or free-text? The plan assumes a small controlled set so Designer can style all badges — confirm with Mona.
2. **Priority vocabulary:** `High`/`Medium`/`Low` vs. numeric (`P0`/`P1`/`P2`)? The plan assumes the three-word set.
3. **Additional optional fields:** Mona's brief mentions "a short contributor-friendly summary" — should we add an optional `summary` field to each project? Not required by CI, but useful. Recommend yes; confirm.
4. **Launch runtime:** Any preference between `type: "node"` shim vs. a `debugpy`/tasks-based approach? CI only checks JSON validity + the name + `index.html`; the plan defaults to whatever cleanest strict-JSON form Coder chooses.
5. **Port:** Step 3 prompt pins `python3 -m http.server 5500`; if 5500 is unavailable in a Codespace, is a fallback port acceptable? Recommend keeping 5500 as documented.
6. **Data source of truth:** Should sample project data reflect real repos on Mona's team, or remain illustrative placeholders? Assume illustrative.
