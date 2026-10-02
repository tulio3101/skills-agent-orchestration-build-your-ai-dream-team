# Project Pulse implementation plan

## Goal and outcome

Build a small, polished static dashboard that lets contributors quickly see
which projects are active, who owns them, their current status, recent
activity, and priority or risk. The first view must be the Project Pulse UI,
not a server directory listing. The implementation will use the existing
repository layout and the four required deliverables:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

The dashboard should present multiple project cards with clear hierarchy,
status badges, priority treatment, readable spacing, responsive behavior, and
accessible markup. It should remain a dependency-free static app that can be
previewed from VS Code.

## Agent responsibilities and file ownership

| Agent | Assigned responsibility | Files |
| --- | --- | --- |
| **Orchestrator** | Coordinate the phases, pass the plan to the specialists, enforce the file boundaries, integrate their reports, and run the final validation. | Coordination only; no application/config source changes |
| **Planner** | Research the brief, workflow requirements, existing agent guidance, dependencies, risks, and validation criteria; maintain this implementation plan. | `docs/project-pulse-plan.md` |
| **Designer** | Define the information hierarchy and visual system, then implement the polished responsive and accessible styling. Specify the markup hooks the Coder must preserve, including visible cards, badges, priority treatment, focus states, contrast, and reduced-motion-friendly behavior where applicable. | `app/styles.css` |
| **Coder** | Implement the semantic page structure, load the data, render visible project cards, provide the required project fields in deterministic JSON, and create the runnable VS Code configuration. Keep the app dependency-free and handle data-loading/rendering failures visibly rather than silently. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |

The Designer owns CSS so visual work does not conflict with the Coder's
markup/data work. The Coder owns the HTML hooks consumed by the stylesheet and
must use the Designer's agreed selectors rather than inventing competing
structure. The Coder also owns the launch configuration because it is runnable
app support and must be validated against the exact server behavior.

## Ordered implementation phases

### 1. Confirm requirements and interfaces

**Owner:** Orchestrator, with Planner input.

Before implementation, confirm the following interfaces from the brief and
workflow:

- `app/project-data.json` has a top-level `projects` array.
- Every project object has `name`, `owner`, `status`, `recentActivity`, and
  `priority`.
- `app/index.html` has the exact title `Project Pulse`, references
  `styles.css` and `project-data.json`, and renders one `.project-card` per
  project with status, recent activity, and priority visible.
- `app/styles.css` defines `.dashboard` and `.project-card`, plus polished
  rounded, shadowed, responsive card styling.
- `.vscode/launch.json` is strict JSON, has the name `Run Project Pulse
  Dashboard`, serves from the `app/` directory with `python3 -m http.server
  5500`, and opens `http://localhost:%s/index.html`.

### 2. Establish the design contract

**Owner:** Designer. **Output:** design decisions and selector contract for
the Coder, then `app/styles.css`.

Define a contributor-first page hierarchy: a dashboard heading and short
context followed by a responsive collection of project cards. Each card should
make the project name and owner easy to scan, use a status badge with readable
text (not color alone), expose recent activity as supporting detail, and make
priority/risk visually prominent without reducing contrast.

Implement the styling in `app/styles.css` with clear typography, spacing,
contrast, `border-radius`, `box-shadow`, responsive layout behavior, keyboard
focus visibility, and sensible small-screen wrapping. Preserve stable
selectors such as `.dashboard` and `.project-card`; document any additional
class hooks in the handoff to the Coder. Avoid adding frameworks, external
assets, or dependencies.

### 3. Implement content, data, and runtime support

**Owner:** Coder.

Create representative data for multiple projects in
`app/project-data.json`. Keep values contributor-friendly and ensure all
required keys exist for every item.

Create `app/index.html` with semantic landmarks, the exact `Project Pulse`
title, a clear empty/loading or error state if needed, and the script needed
to fetch the adjacent JSON file and render project cards. The rendered output
must use the Designer's selectors and visibly expose each project's `status`,
`recentActivity`, and `priority`; do not rely on hover-only or color-only
communication. Keep the page functional when served from `/app/`, rather than
opening the JSON directly from the filesystem.

Create `.vscode/launch.json` as strict JSON with no comments. Set `cwd` to
`${workspaceFolder}/app`, run `python3 -m http.server 5500`, and configure
`serverReadyAction` to open
`http://localhost:%s/index.html`. This must open the dashboard frontend instead
of the directory root.

### 4. Integrate and validate

**Owner:** Orchestrator, with Designer and Coder reports.

Review all four deliverables together. Check that every HTML selector used by
the Coder exists in the stylesheet, every required data field is rendered,
the JSON path is correct relative to the page, and the launch working
directory and URL agree. Resolve integration issues within the owning agent's
file scope rather than making overlapping edits.

## Dependencies and parallel-work decisions

The work is intentionally split around file ownership:

1. Requirement/interface confirmation must happen first so the agents share
   the exact selectors, data schema, launch name, command, and URL.
2. After confirmation, the Designer may work on `app/styles.css` in parallel
   with the Coder drafting `app/project-data.json` because those files have no
   overlapping edits or data dependency.
3. The Coder's `app/index.html` depends on the Designer's selector contract
   and the agreed JSON schema. The Coder may scaffold semantic markup while
   the Designer works, but final HTML selector choices and rendering
   integration must be checked after the design contract is available.
4. `.vscode/launch.json` can be created in parallel with the app files after
   the required launch settings are confirmed; it does not depend on the
   visual design, but end-to-end browser validation must wait until
   `app/index.html` exists.
5. Integration and browser validation are sequential after both specialists
   finish. Do not have Designer and Coder edit the same file concurrently, and
   do not validate a launch configuration before the app target is present.

## Edge cases and implementation risks

- Opening `http://localhost:5500/` may show a directory listing; the launch
  URL must explicitly end in `/index.html`.
- Fetching JSON directly from `file://` can fail due to browser restrictions;
  validation must use the configured HTTP server.
- Missing, malformed, or empty project data should produce a visible,
  understandable state rather than an empty success-looking dashboard.
- Required fields may contain long text; cards and badges must wrap without
  clipping or horizontal overflow.
- Status and priority colors must maintain readable contrast and be
  understandable without color perception.
- Keyboard users need visible focus treatment, and the responsive layout must
  remain usable at narrow widths.
- `launch.json` must remain valid JSON with no comments or trailing syntax
  that the JSON parser rejects.

## Validation expectations

### Static and repository checks

Run the repository's `scripts/validate-exercise.sh` to ensure existing
workflow/configuration checks still pass. Then run targeted checks for the
new artifacts:

- `python3 -m json.tool app/project-data.json`
- `python3 -m json.tool .vscode/launch.json`
- Confirm the launch configuration name, `cwd`, command, port, and
  `serverReadyAction` URL exactly match the brief.
- Confirm `app/index.html` contains `Project Pulse`, links `styles.css`,
  references `project-data.json`, includes `.project-card`, and contains
  visible rendering paths for `status`, `recentActivity`, and `priority`.
- Confirm `app/styles.css` contains `.dashboard`, `.project-card`,
  `border-radius`, and `box-shadow`.
- Confirm every object in `projects` contains all five required fields.

### Runtime and accessibility checks

Start the equivalent configured server from `app/` with
`python3 -m http.server 5500`, request
`http://localhost:5500/index.html`, and verify the response is the dashboard
document rather than a directory listing. In a browser, use the **Run Project
Pulse Dashboard** launch configuration and confirm that multiple project
cards appear with the project name, owner, status, recent activity, and
priority. Check the narrow layout, keyboard focus order, readable contrast,
text wrapping, and that the UI remains understandable without relying on
color alone. Stop the preview server after validation.

The Orchestrator's final report should identify the Planner, Designer, Coder,
and Orchestrator contributions, state which checks passed, and call out any
remaining limitations without claiming runtime validation that was not
performed.
