# Tooling Matrix

Use this reference when architecture discovery would benefit from local tools for dependency mapping, dead-code checks, SAST, diagrams, runtime traces, database introspection, or repository packaging.

## License-safe defaults

Default architecture discovery must stay local, free, and open-source. Do not ask agents or users to install, configure, or rely on paid SaaS, commercial-only features, license-restricted scanners, or hosted project introspection unless the user explicitly allows that scope.

Default does not mean "always run." Use these tools only when relevant to the discovered stack and when execution or installation is within the user's allowed scope.

| Status | Tools | Use |
| --- | --- | --- |
| `default` | `dependency-cruiser` | Primary JS/TS dependency-boundary and import graph evidence. MIT licensed. |
| `default` | `knip` | JS/TS unused files, dependencies, and exports. ISC licensed. |
| `default` | Mermaid | Default diagram source format for reports. MIT licensed. |
| `default` | Semgrep CE | Local SAST and pattern scanning when security-sensitive architecture boundaries need evidence. LGPL 2.1. Do not use Pro or Platform-only features by default. |
| `default` | Playwright | Runtime route and UI traces when app execution is allowed. Apache-2.0 licensed. |
| `default` | Repomix | Bounded Codex review packaging when useful, with strict exclusions for secrets, dependency folders, build output, logs, coverage, and generated artifacts. MIT licensed. |
| `conditional` | `madge` | Fallback or secondary graph tool when dependency-cruiser is unavailable or a quick visual import graph helps. MIT licensed. |
| `conditional` | `@nestjs/swagger` | Use only when NestJS is detected and OpenAPI generation is already appropriate for the app. MIT ecosystem. |
| `conditional` | Compodoc | Use for Angular, NestJS, or TypeScript documentation evidence when it fits the repo. MIT licensed. |
| `conditional` | Supabase CLI | Use only for local CLI evidence, local Supabase, migrations, or local/read-only Postgres introspection. MIT licensed CLI. |
| `optional/non-default` | Structurizr DSL, CLI, Lite | Use only when the user specifically wants C4/model-as-code output or the repo already uses Structurizr. Keep Mermaid as the default diagram path. CLI is Apache-2.0; CLI/Lite are end-of-life and paid/cloud/server offerings exist. |
| `optional/non-default` | Opengrep | Optional Semgrep-compatible SAST alternative for users who prefer a more open-governance scanner. LGPL 2.1. |
| `do-not-default` | CodeQL | Use only when already configured in the repo or when the user explicitly confirms it is allowed/licensed for the target. Do not make CodeQL part of the default architecture workflow. |

## Supabase and database introspection

Prefer evidence in this order:

1. Existing migrations, schemas, seed files, generated types, and local configuration.
2. Local Supabase or local Postgres commands.
3. `psql` read-only introspection against an explicitly approved database.
4. Linked or hosted Supabase project commands only with explicit user approval.

Do not run migrations, seeds, writes, destructive commands, or hosted project introspection by default.

## Security scanning

Use Semgrep CE by default only for local, open-source scanning. Avoid commercial Semgrep features, cloud uploads, managed policies, and platform workflows unless the user explicitly asks for them. Treat CodeQL as non-default because private or general-purpose use can depend on GitHub Code Security licensing.

## Diagrams

Use Mermaid source as the default report diagram format. Keep Structurizr optional for C4 or existing Structurizr workflows, and include generated DSL only when that is part of the requested deliverable.

## License references

- dependency-cruiser: <https://github.com/sverweij/dependency-cruiser>, <https://github.com/sverweij/dependency-cruiser/blob/main/LICENSE>
- madge: <https://github.com/pahen/madge>
- knip: <https://github.com/webpro-nl/knip>, <https://www.npmjs.com/package/knip>
- NestJS OpenAPI: <https://docs.nestjs.com/openapi/introduction>
- Compodoc: <https://github.com/compodoc/compodoc>
- Supabase CLI: <https://github.com/supabase/cli>
- Mermaid: <https://github.com/mermaid-js/mermaid/blob/develop/LICENSE>
- Structurizr CLI: <https://github.com/structurizr/cli>
- Structurizr Lite: <https://docs.structurizr.com/lite>
- GitHub CodeQL CLI: <https://docs.github.com/en/code-security/concepts/code-scanning/codeql/codeql-cli>
- Semgrep CE: <https://semgrep.dev/products/community-edition/>
- Opengrep: <https://github.com/opengrep/opengrep>
- Playwright: <https://github.com/microsoft/playwright>
- Repomix: <https://repomix.com/>
