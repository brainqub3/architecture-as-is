# Architecture As Is

[![skills.sh](https://skills.sh/b/brainqub3/architecture-as-is)](https://skills.sh/brainqub3/architecture-as-is/as-built-architecture)

An agent skill for exploring the architecture of an existing codebase as it actually exists.

This is designed for inherited, vibe-coded, legacy, prototype, or poorly documented repositories where you need a factual map before making decisions.

## Skill

`as-built-architecture`

The skill guides an agent to:

- inspect the repository structure, manifests, entrypoints, scripts, tests, data models, and integrations
- run safe existing checks where appropriate
- avoid polluting the target codebase with stray tests or probe files
- separate observed facts from inferred claims and unknowns
- trace representative user or system flows
- produce an evidence-backed as-built architecture report

## Install

```powershell
npx skills add brainqub3/architecture-as-is --skill as-built-architecture
```

To list available skills in this repository:

```powershell
npx skills add brainqub3/architecture-as-is --list
```

To preview the skill without installing it:

```powershell
npx skills use brainqub3/architecture-as-is@as-built-architecture
```

## When To Use

Use this when you need to answer questions like:

- What architecture has actually been built?
- Which components, entrypoints, services, jobs, and data stores exist?
- What do the tests prove, mock, skip, or omit?
- Where do docs and code disagree?
- Which flows are verified, inferred, or still unknown?
- What should we verify before refactoring, rewriting, onboarding, or extending the system?

## Output

The skill produces a structured report covering:

- scope and assumptions
- executive summary
- as-built system map
- entrypoints and runtime behavior
- data stores and external integrations
- test suite interpretation
- representative flows
- observed, inferred, and unknown architecture claims
- risks and friction
- command log
- next verification steps

## Repository Layout

```text
.agents/
  skills/
    as-built-architecture/
      SKILL.md
```

## Safety

The skill is intentionally read-mostly. It tells the agent not to refactor, create new tests, update snapshots, run migrations, touch production systems, expose secrets, or clean up generated artifacts unless the user explicitly asks.

Review any skill before use. Skills run with the permissions of the agent that loads them.
