# npm Script Convention

Names for the scripts in `package.json`, so that the same task has the same name in every project.
`deploy:prod` means the same thing in every repository, and a tool or an AI assistant can rely on it.

This convention covers the names. How a project is laid out is in [Project Structure Convention](../project-structure-convention/README.md).

## Rules

1. **Lowercase, colon-separated.** `area:action` or `area:action:target`. Use hyphens inside a word (`db:rotate-password`).
2. **Area first, action second, target last.** `db:backup:prod`, not `backup-prod-db`.
3. **Target is one of** `dev`, `test`, `preprod`, `prod`, in promotion order. Nothing else is an environment.
4. **Maximum three parts.** If a name needs a fourth part, the area is too broad: split it.
5. **Universal names are fixed.** Use them with exactly these spellings, and don't add synonyms (`serve`, `start:dev`).
6. **Project-specific names** (generators, importers, one-off tools) use an area that describes the project's own thing, such as `generate:entities` or `img:convert`. They never reuse a universal name.
7. **Every project gets** `prepare` (hook setup), `lint`, `typecheck`, `test`, `build`, `dev`.

## Targets

Each target is a separate environment with its own server, database and env parameters in SSM (`/env/<project>/env.<target>`).

| Target | Meaning | Used by |
|---|---|---|
| `dev` | Developer's own or shared integration environment | developers |
| `test` | QA and automated tests on a shared server | QA, CI |
| `preprod` | Staging with the same setup as production; last check before a release | release |
| `prod` | Production | users |

Changes move in this order: `dev` → `test` → `preprod` → `prod`. A project may skip a target it does not have.

A task that runs only on this machine has no suffix (`dev` runs the app locally). A task that runs against a server takes the target suffix (`db:backup:prod`).

## Universal names

| Script | Meaning |
|---|---|
| `dev` | Run locally with watch and reload |
| `start` | Run the built or production entry point |
| `build` | Produce build output |
| `lint` | Static checks (formatter and linter in check mode) |
| `lint:fix` | Apply automatic fixes from `lint` |
| `typecheck` | Type checks without emitting files |
| `test` | Unit tests (fast, no network) |
| `test:integration` | Tests that need running services |
| `prepare` | Install the git hooks (`git config core.hooksPath .githooks`), tolerant of a missing `.git` |

## Area names

| Area | Actions and targets | Example |
|---|---|---|
| `deploy` | `:dev`, `:test`, `:preprod`, `:prod` | `deploy:preprod` |
| `db` | `:migrate`, `:migrate:dev`, `:backup:<target>`, `:rotate-password:<target>`, `:sync:<from>-to-<to>` | `db:sync:test-to-preprod` |
| `release` | `:patch`, `:minor`, `:major` | `release:minor` |
| `env` | `:sync` (pull the env file from SSM) | `env:sync` |
| `test` | `:integration` | `test:integration` |

Notes on the table:

- `db:migrate` applies the committed migrations to the database the project is configured for. `db:migrate:dev` creates a new migration from schema changes and is for local development only.
- `db:sync:<from>-to-<to>` overwrites the target database. It backs up the target first and asks for confirmation. Syncing into `prod` is allowed only from `preprod`, never directly from `dev` or `test`.
- `db:rotate-password:<target>` changes a database user's password and its SSM parameter, and updates the app on the same host.
- `release:*` bumps the version, writes the changelog and publishes.

## Naming a new script

1. Find the area in the table. If it fits, use the next action in that area.
2. If it does not fit, pick a new area, keep it to one word, and add it to this document in a pull request.
3. Put the target last, and only one of the four targets from the table.
4. Check the name against rule 5 before adding it.

## Open decisions

- **Our shared server.** Today drh-mini is the only shared server, and scripts call it `:dev`. It plays the `test` role, so the convention says `:test`. Renaming means `deploy:dev` → `deploy:test` and `db:*:dev` → `db:*:test` in every project, plus the deploy workflow's `target` choices. Until that is decided, read `:dev` in project scripts as "the shared server".
- **`preprod`.** We have no staging server yet. The name is reserved; add it when a server exists.

## Deviations found in current projects

These names do not follow the convention yet. Rename them in a separate commit per project.

| Project | Current | Convention |
|---|---|---|
| all with a shared server | `deploy:dev`, `db:*:dev` (shared server) | `:test` (see Open decisions) |
| dungeons-www | `serve` | `dev` |
| dragons-legends-core-cdn | `dev:local` | `dev` (the local-only mode is documented in its README) |
| dungeons-api | `user:set-password`, `test-users:create`, `test-users:delete` | project-specific area, e.g. `test-user:create` (decide per project) |
| freetor-api | `create-user`, `server:setup` | project-specific; keep, but document in the project README |
| dragons-legends-core-api | `version` | remove, the release scripts handle versioning |
| dragons-legends-core-cdn | `img:backup:dev`, `img:sync:*` | fine; already `area:action:target` |

## Examples

```json
{
  "scripts": {
    "dev": "bun --watch src/index.ts",
    "start": "bun src/index.ts",
    "build": "bun build.ts",
    "lint": "biome lint src",
    "typecheck": "tsc --noEmit",
    "test": "bun test",
    "prepare": "git config core.hooksPath .githooks 2>/dev/null || true",
    "db:migrate": "bunx prisma migrate deploy",
    "db:backup:prod": "hlpr mariadb backup -ssm-param /env/<project>/env.prod",
    "db:rotate-password:prod": "bun ./node_modules/@tertium/js/scripts/db-rotate-password/db-rotate-password.ts --ssm-param=/env/<project>/env.prod",
    "deploy:test": "bun ./node_modules/@tertium/js/scripts/workflow/workflow.ts run deploy -f target=test",
    "deploy:preprod": "bun ./node_modules/@tertium/js/scripts/workflow/workflow.ts run deploy -f target=preprod",
    "deploy:prod": "bun ./node_modules/@tertium/js/scripts/workflow/workflow.ts run deploy -f target=prod",
    "release:patch": "gith release --patch",
    "release:minor": "gith release --minor",
    "release:major": "gith release --major"
  }
}
```

## For AI assistants

- Use the names in this document. Do not invent a new name when one of the table entries fits.
- Use the universal names exactly. Never add `serve` or `start:dev`.
- Targets are `dev`, `test`, `preprod`, `prod`. Promotion goes in that order.
- Until the Open decisions are settled, say "the shared server" for the server behind `:dev` when you talk to a person.
