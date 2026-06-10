---
name: as-built-architecture
description: Use this skill whenever the user asks to understand, map, audit, reverse-engineer, document, or reason about the architecture of an existing codebase, especially a vibe-coded, prototype, legacy, inherited, or poorly documented repo. This skill guides an agent to discover the architecture as it actually exists, not as it was intended, by combining static inspection, dependency and entrypoint discovery, safe execution, evidence-backed flow tracing, upfront scoping questions, and an as-built architecture report.
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

Use language-specific tooling where it exists, but keep the findings evidence-backed.

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

Use diagrams when they reduce ambiguity. Mermaid is usually enough. Label diagrams as "as-built" and avoid showing planned or desired architecture unless the user separately asks for it.

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

## Report structure

Use this structure unless the user requests a different output.

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
- Do not clean up files, reset git state, or remove generated artifacts.
- Do not assume conventional architecture just because a framework is present.
- Do not hide command failures; they are useful architecture evidence.
- Do not expose secrets.
- Do not turn the report into a wishlist. Keep the primary artifact grounded in the current system.
