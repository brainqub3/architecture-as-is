---
name: as-built-architecture
description: Use this skill whenever the user asks to understand, map, audit, reverse-engineer, document, or reason about the architecture of an existing codebase, especially a vibe-coded, prototype, legacy, inherited, or poorly documented repo. This skill guides an agent to discover the architecture as it actually exists, not as it was intended, by combining static inspection, dependency and entrypoint discovery, safe execution, evidence-backed flow tracing, upfront scoping questions, and a timestamped readable HTML as-built architecture report.
---

# As-Built Architecture Discovery

Use this skill to build a factual architecture snapshot of an existing codebase. The goal is not to improve, refactor, or redesign the system. The goal is to make the current system understandable enough that the user can reason about what has been built.

## Core stance

Treat the repository as the source of truth. Existing documentation, file names, framework conventions, and user expectations are useful clues, but code and runtime behavior take priority.

Keep a strict separation between:

- **Observed**: directly seen in files, commands, tests, logs, configs, schemas, or runtime behavior.
- **Inferred**: likely based on multiple observations, but not directly proven.
- **Unknown**: not verified, blocked, missing, ambiguous, or outside the safe exploration scope.

Do not present intended architecture as fact. If a README claims one thing and code suggests another, report the mismatch explicitly.

## Start with narrow questions

Ask up to three questions only when the answers would materially change scope, risk, or cost. Do not block on questions if reasonable assumptions let you begin safely.

Good upfront questions:

1. What decision should this architecture snapshot support: onboarding, rewrite planning, risk audit, feature work, production debugging, or something else?
2. What scope matters most: the whole repo, one app/package, one user flow, one service, or one suspected problem area?
3. What execution is allowed: installing dependencies, running tests/builds, starting local services, using Docker, touching databases, or making network calls?

If the user does not answer, assume:

- Scope is the current workspace or repo root.
- Exploration should be read-mostly and non-destructive.
- Existing tests, typechecks, builds, and script-listing commands are allowed.
- Migrations, data writes, deploys, secret-revealing commands, destructive cleanup, and production network calls require explicit approval.

## Exploration workflow

### 1. Establish the repository map

Start broad, then narrow. Prefer fast file discovery and direct evidence.

- Check working tree state with `git status --short` so user changes are not confused with architecture.
- List files with `rg --files` or the fastest available equivalent.
- Read top-level docs, manifests, lockfiles, workspace configs, build configs, Docker files, CI files, env examples, and package manager metadata.
- Identify languages, package managers, frameworks, apps, packages, generated folders, and likely monorepo boundaries.
- Note stale or contradictory documentation instead of trusting it.

### 2. Identify entrypoints and runtime surfaces

Find how the system starts and how work enters it.

- App servers, route handlers, controllers, pages, API endpoints, RPC handlers, CLIs, workers, cron jobs, queue consumers, event handlers, test harnesses, and scripts.
- Build, dev, test, lint, typecheck, seed, migrate, and deploy commands.
- Frontend routes, backend routes, public APIs, internal APIs, and background paths.
- Runtime configuration: ports, env vars, config files, service names, container definitions, and process managers.

Capture both obvious entrypoints and hidden ones. A small script in `package.json`, a worker file, or a CI-only command can reveal real architecture.

### 3. Map dependencies and internal boundaries

Use manifests and imports to understand the real module graph.

- External dependencies and what they imply: web framework, ORM, auth, state management, queue, storage, logging, testing, build tooling.
- Internal package boundaries, shared libraries, utility modules, generated clients, and cross-package imports.
- Cycles, god modules, duplicated abstractions, framework leakage, and direct data access from unexpected layers.
- Code paths that bypass the apparent architecture, such as direct database calls in UI routes or business logic inside controllers.

Use language-specific tooling where it exists, but keep the findings evidence-backed. Before using tools for dependency maps, unused-code checks, SAST, diagrams, runtime traces, database introspection, or repository packaging, read `references/tooling-matrix.md` and apply its license-safe defaults.

### 4. Run safe commands to test runtime reality

Prefer existing project commands over invented commands.

- List available scripts and tasks before running expensive commands.
- Run lightweight checks first: dependency metadata, unit tests for targeted packages, typecheck, lint, build, or framework route listings when available.
- If starting a local server is safe, capture startup command, port, logs, route output, and shutdown cleanly.
- Record failures as useful facts. A failing build, missing env var, broken install, or test suite that cannot run is part of the as-built architecture.

Do not run migrations, seeds, destructive cleanup, production deploys, or commands that may write to external systems unless the user explicitly allows it.

### 5. Reason about tests without polluting the codebase

Existing tests are architecture evidence. They reveal supported flows, boundaries, fixtures, fake services, data factories, and what prior developers considered important. Use them to understand the system, but do not create stray tests or probe files in the target repo.

- Discover test frameworks, commands, config files, test directories, fixtures, factories, mocks, snapshots, and CI test steps.
- Map tests to architecture: which components have coverage, which flows are exercised, which layers are mocked, and which runtime surfaces are untested.
- Read representative tests as behavior documentation, especially integration, end-to-end, contract, and regression tests.
- Treat tests as evidence with care. A test can prove an intended behavior path exists, but mocks, skipped tests, stale snapshots, test-only configs, or fake data may not match production behavior.
- Prefer running existing targeted tests over full suites when the repo is large or risky. Record skipped, flaky, failing, or non-runnable tests as architecture facts.
- Check `git status --short` before and after test/build commands. If commands generate caches, coverage, snapshots, reports, build output, or other artifacts, report them in the command log instead of silently cleaning or committing them.
- Do not add new test files, snapshots, fixtures, or committed probes to the codebase unless the user explicitly asks for verification tests to be written.
- If a temporary probe is necessary, prefer command-line one-liners, REPL commands, dry-run modes, or a temporary directory outside the repo. Record the probe and remove temporary external artifacts when practical.
- Do not update snapshots, approve golden files, rewrite fixtures, or change test configuration during architecture discovery.

Use test results to qualify confidence:

- **High confidence**: static evidence and an existing test or runtime check agree.
- **Medium confidence**: static evidence exists, but tests are mocked, skipped, stale, or cannot run.
- **Low confidence**: the architecture claim comes only from test names, docs, or inferred conventions.

### 6. Understand state, data, and external systems

Architecture is often hidden in persistence and integration boundaries.

- Locate database schemas, migrations, ORM models, repositories, API clients, generated types, data loaders, caches, queues, object storage, auth providers, email providers, payment clients, analytics, feature flags, and scheduled tasks.
- Identify which component appears to own each major data concept.
- Note where data contracts are implicit, duplicated, untyped, or split across layers.
- Do not print secrets. If env files contain credentials, mention the presence of required configuration without exposing values.

### 7. Trace representative behavior slices

Pick two to four high-value flows based on the user's goal and the discovered entrypoints. For each flow, trace from entrypoint to side effects.

Examples:

- Browser route -> component -> API call -> server handler -> service -> database.
- Webhook -> validation -> queue -> worker -> external API -> status update.
- CLI command -> parser -> business operation -> filesystem/database output.

For each flow, record:

- Entry condition.
- Main files/functions involved.
- Data read and written.
- External calls.
- Error handling and fallback behavior.
- Evidence and confidence.

### 8. Build the as-built model

Synthesize only after gathering evidence.

Describe the current architecture in terms of:

- Components and responsibilities.
- Runtime processes.
- Entrypoints.
- Data stores and state ownership.
- External services.
- Cross-component communication.
- Shared abstractions.
- Deployment and operational assumptions.
- Test and build coverage.

Use diagrams when they reduce ambiguity. Author diagrams as Mermaid by default, and keep each diagram's `.mmd` source under the run folder's `diagrams/` subfolder, unless the user asks for another format or the repo already has a diagram-as-code convention. Mermaid source is plain text: it only becomes a picture when a renderer turns it into SVG, so the diagram that lands in the HTML report must be a pre-rendered, inlined `<svg>` (see "Diagrams in the HTML report"), not raw Mermaid source presented as a finished diagram. When no renderer is in execution scope, fall back to a clearly labeled unrendered-source block or an ASCII text diagram. Label diagrams as "as-built" and avoid showing planned or desired architecture unless the user separately asks for it.

### 9. Surface implications, not redesigns

The user needs to reason about what exists. Identify implications without turning the report into a refactor plan.

Call out:

- Hidden coupling.
- Unclear ownership.
- Mismatches between docs, scripts, and code.
- Missing or non-running tests.
- Environment assumptions.
- Dead, duplicate, or unreachable-looking code.
- Security-sensitive boundaries.
- Production-readiness gaps.
- Places where architecture depends on convention instead of explicit contracts.

When suggesting next steps, keep them verification-oriented unless the user asks for remediation.

## Artifact hygiene

Keep architecture discovery artifacts organized and easy to review. All artifacts created by this skill must live under one run folder:

```text
architecture_as_is/
  YYYYMMDD_HHMMSS/
    architecture_as_is.html
    manifest.md
    evidence/
    diagrams/
    exports/
    screenshots/
```

Rules:

- Create one timestamped run folder per architecture discovery run, using local machine time in `YYYYMMDD_HHMMSS` format.
- Save the primary HTML report as `architecture_as_is/YYYYMMDD_HHMMSS/architecture_as_is.html`.
- Create `manifest.md` for every run. List generated files, why each exists, the command or tool that produced it, and whether it is intended to be committed, ignored, or reviewed and deleted.
- Put optional evidence files such as command logs, inventories, route listings, and summarized trace notes under `evidence/`.
- Put diagram sources under `diagrams/`: Mermaid `.mmd`, text diagrams, C4 DSL, and any rendered `.svg` produced from them for inlining into the report.
- Put machine-readable tool output, dependency graphs, scanner summaries, and packaged context under `exports/`.
- Put runtime screenshots or visual trace images under `screenshots/`.
- Create optional subfolders only when needed. Do not create empty folders just to satisfy the template.
- Do not write architecture artifacts outside the run folder unless the user explicitly asks for a different destination.
- Do not overwrite older runs. If a report is regenerated, create a new timestamped run folder.
- Do not include secrets, raw credentials, dependency folders, build output, coverage folders, giant raw logs, or unbounded scanner output in architecture artifacts.

Artifacts produced incidentally by existing project commands, such as framework caches or build outputs, are not architecture artifacts. Do not move or clean them silently; record them in the report and manifest.

## Report structure

Use this structure unless the user requests a different output. The final report must be written as a readable, self-contained HTML file.

```markdown
**Scope And Assumptions**
- Scope explored:
- Execution allowed/performed:
- Important assumptions:
- Not explored:

**Executive Summary**
- 5-8 bullets describing what is actually built.

**As-Built System Map**
| Component | Current responsibility | Key evidence | Confidence |
| --- | --- | --- | --- |

**Entrypoints And Runtime**
- Commands:
- Apps/services/workers:
- Routes/APIs/CLIs/jobs:
- Runtime configuration:

**Data And Integrations**
- Data stores:
- Schemas/models:
- External systems:
- Ownership and contracts:

**Test Suite Interpretation**
- Test frameworks and commands:
- What tests prove:
- What tests mock or omit:
- Failing/skipped/non-runnable tests:
- Artifacts or workspace changes observed:

**Representative Flows**
1. Flow name
   - Path:
   - Reads/writes:
   - Side effects:
   - Evidence:
   - Confidence:

**Architecture Observations**
- Observed:
- Inferred:
- Unknown:

**Risks And Friction**
| Finding | Why it matters | Evidence | Suggested verification |
| --- | --- | --- | --- |

**Command Log**
| Command | Purpose | Result |
| --- | --- | --- |

**Next Verification Steps**
- Short list of the highest-value follow-up checks.
```

## HTML report output

At the end of every architecture discovery run, create a readable HTML report that details the architecture. This report is the primary deliverable.

- Save the run under a folder named `architecture_as_is` at the explored repository root. If the repository root is uncertain, use the current working directory and state that assumption.
- Create a timestamped run folder if it does not already exist.
- Put the timestamp on the run folder using local machine time in `YYYYMMDD_HHMMSS` format.
- Use this primary report filename pattern: `architecture_as_is/YYYYMMDD_HHMMSS/architecture_as_is.html`.
- Write the run manifest as `architecture_as_is/YYYYMMDD_HHMMSS/manifest.md`.
- Make the HTML self-contained: embed CSS in a `<style>` block, embed any diagrams as inline `<svg>`, and do not require external assets, CDNs, build tools, JavaScript, or network access to read the report. "Self-contained" means the report must render correctly when opened from a local file with the network disconnected and scripts disabled. This rules out a CDN `<script src>` that pulls a diagram runtime, and it also rules out hand-inlining the multi-hundred-KB Mermaid JavaScript bundle with a `mermaid.initialize`/`run` call. The supported way to show a diagram is to pre-render it to static SVG and paste that `<svg>` inline (see "Diagrams in the HTML report").
- Keep the report easy to scan: include a title, generation timestamp, scope, table of contents, summary cards or tables, clear section headings, evidence tables, confidence labels, and command log.
- Convert the "Report structure" template into real HTML, not pasted Markdown. The template above is written in Markdown for readability, but the report is an HTML document where Markdown syntax is not special. Pipe tables must become `<table>`/`<tr>`/`<th>`/`<td>`, `**bold**` headers must become `<h2>`/`<strong>`, numbered and bulleted lists must become `<ol>`/`<ul>`/`<li>`, and fenced blocks must become `<pre><code>`. Never paste literal `|`-and-`---` table syntax or `**` markers into the HTML body; in a browser they show as plain-text characters instead of tables and headings.
- Include the same substantive sections as the report structure above.
- Escape code snippets, command output, file paths, and user-provided text before inserting them into HTML. This applies to text regions only. Do not HTML-escape the markup of an inlined `<svg>` element — escaping its tags and attributes would corrupt the vector geometry and stop it rendering; paste the `<svg>` verbatim. When you fall back to showing Mermaid source as text in a `<pre>`/`<details>` block, that source is text and must be HTML-escaped (escape `&` first, then `<` and `>`) so Mermaid arrow syntax round-trips when copied back into a renderer.
- For every diagram, follow "Diagrams in the HTML report" below: keep the `.mmd` source under `diagrams/`, render it to static SVG and inline that SVG when render tooling is in scope, and otherwise use the labeled-source or ASCII fallback. Never present raw Mermaid source as a finished diagram.
- Run `git status --short` before and after writing architecture artifacts. Report any generated or changed files in the command log, manifest, and final response.

### Diagrams in the HTML report

Mermaid source is plain text; it only becomes a visible diagram when a renderer converts it to SVG. Because the report must be self-contained, offline, and JavaScript-free, render each diagram ahead of time and ship the result as inline SVG, or fall back to a clearly labeled source/ASCII diagram. Choose the path that matches the execution scope the user granted.

**Execution-scope note.** Rendering Mermaid to SVG requires installing tooling and a headless browser. Under this skill's default read-mostly scope (see "Start with narrow questions"), dependency installation is a gated question, not an assumed allowance. So the inline-SVG primary path is only available when the user has granted dependency-install or headless-browser scope. If that scope was not granted, do not silently drop the diagram: use the always-available text fallback and state in the report that diagrams were not rendered because rendering tooling was out of scope.

**Primary path — pre-render to inline SVG (use when render tooling is in scope):**

1. Write each diagram's Mermaid source to its own `.mmd` file under the run folder's `diagrams/` subfolder (for example `diagrams/component-graph.mmd`).
2. Render it to a static `.svg` with the Mermaid CLI (`mmdc`). Use a config that keeps the SVG offline-portable:
   - `htmlLabels: false` and `flowchart.htmlLabels: false` so labels become native SVG `<text>` nodes, not `<foreignObject>` HTML (which renders inconsistently and depends on HTML/CSS).
   - `themeVariables.fontFamily` set to a generic stack such as `ui-sans-serif, system-ui, sans-serif` so the SVG never fetches a webfont.
   - Give every diagram a unique SVG id with `--svgId` so multiple inlined diagrams do not collide (see step 4).
   - Example: `npx -p @mermaid-js/mermaid-cli mmdc -i diagrams/component-graph.mmd -o diagrams/component-graph.svg -b transparent -c diagrams/mermaid-config.json --svgId as-built-component-graph`
3. If installing `mmdc`'s headless-Chromium dependency is out of scope but a local Mermaid install and the Playwright browser (already listed in `references/tooling-matrix.md`) are available, you may instead render in Playwright: load a local copy of `mermaid.js` (from an in-scope local install — never a CDN) in a page and call `mermaid.render()`, then capture the returned SVG string. The bundled Playwright browser alone does not include `mermaid.js`; if you cannot obtain `mermaid.js` offline, do not use this path and do not fetch it from a CDN — use the fallback instead.
4. Make each inlined SVG collision-free. `mmdc` emits the same root `id` and identical internal fragment ids (markers, gradients, clip paths) for every diagram, so two inlined SVGs would share ids — invalid HTML, and later diagrams' `url(#...)` marker references (arrowheads) would bind to the first SVG's defs, producing wrong or missing arrowheads and CSS bleed. Give each diagram a unique id via `mmdc --svgId`, or post-process the SVG to rewrite the root `id` and every internal fragment id and `url(#...)` reference to a per-diagram prefix. No two inlined `<svg>` blocks may share an id or a `def`/marker id.
5. Read the generated `.svg`, drop the XML prolog/doctype, keep the `<svg>...</svg>` root (retain its `xmlns` attribute), and paste that markup verbatim into the HTML body inside a `<figure>` with a `<figcaption>`. Do not HTML-escape the SVG markup.
6. Before shipping, verify each inlined SVG is offline-clean: no `<script>`, no `<foreignObject>`, no `@font-face` or `@import`; the only `http(s)` strings are the W3C `xmlns` namespace URIs (identifiers, not fetches); and every `url(#...)` resolves to a `def` within the same SVG after id uniquification.
7. Sanitize any repo- or user-derived text used in diagram labels before it reaches the `.mmd`/SVG. Keep diagram styles inside the `<svg>` or the report `<style>`, never an external stylesheet. Record the `.mmd` source, the `mermaid-config.json`, the rendered `.svg`, and the render command in `diagrams/`, the manifest, and the command log.

**Fallback path — always available, no tooling (use when no renderer is in scope):**

1. Provide a simple ASCII box-and-arrow diagram inside a `<pre>` so the report still conveys structure with zero dependencies. This is the baseline that always works.
2. Optionally also keep the `.mmd` under `diagrams/` and include the HTML-escaped Mermaid source in a clearly labeled `<pre class="mermaid-source">` (optionally inside `<details>`), with a one-line note that it is unrendered Mermaid source because rendering was out of scope and the reader can paste it into any Mermaid renderer to view it.
3. When escaping `.mmd` source for the `<pre>`, escape `&` first, then `<` and `>`, so Mermaid arrow syntax (`-->`, `---`, `&`) round-trips correctly when copied back into a renderer.
4. Do not inline a Mermaid runtime and do not reference a CDN to "fix" the fallback; the labeled-source/ASCII form is the intended offline-safe result.

**Hard rules — these are the current bug; never do them:**

- Never put a bare ```` ```mermaid ```` fenced block in the HTML. A Markdown fence is not special inside an HTML document, so it renders as literal backticks and source text, not a diagram.
- Never put a `<pre class="mermaid">` (or any element holding Mermaid source) into the HTML styled or captioned as a finished diagram. The runtime that would transform it is forbidden and never present, so it would stay inert text. Any Mermaid source that reaches the HTML must be in the fallback block above, explicitly labeled as unrendered source.
- Never reference a diagram via `<img src="diagrams/...">`. An external image reference violates self-containment and breaks if the folder moves. Inline the `<svg>` instead.

The timestamped run folder is the expected default write. Keep the primary HTML report and manifest mandatory; create additional architecture artifacts only when they materially improve reviewability. Do not create probe files, tests, snapshots, or cleanup changes unless the user explicitly asks.

## Evidence standards

- Include file paths and line numbers for important claims when possible.
- Prefer a few strong citations over a long list of weak ones.
- Include command outputs in summarized form; do not dump huge logs.
- Mark confidence as High, Medium, or Low.
- If a claim cannot be verified, say exactly what prevented verification.
- If current library, framework, SDK, CLI, or cloud-service behavior needs documentation, follow the repository's documentation-fetching instructions before answering that part.

## What not to do

- Do not refactor or edit code unless the user explicitly asks.
- Do not create new tests, snapshots, fixtures, or probe files inside the target repo unless the user explicitly asks for them.
- Do not update snapshots, golden files, or test configuration during discovery.
- Do not clean up files, reset git state, or remove generated artifacts. The expected generated artifacts are the timestamped run folder, primary HTML report, and manifest under `architecture_as_is`.
- Do not assume conventional architecture just because a framework is present.
- Do not hide command failures; they are useful architecture evidence.
- Do not expose secrets.
- Do not turn the report into a wishlist. Keep the primary artifact grounded in the current system.
