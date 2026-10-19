# npm Script Convention

Names for the scripts in `package.json`, so that the same task has the same name in every project.
`deploy:dev` means the same thing in every repository, and a tool or an AI assistant can rely on it.

This convention covers the names. How a project is laid out is in [Project Structure Convention](../project-structure-convention/README.md).

## Rules

1. **Lowercase, colon-separated.** `area:action` or `area:action:target`. Use hyphens inside a word (`db:rotate-password`).
2. **Area first, action second, target last.** `db:backup:prod`, not `backup-prod-db`.
3. **Target is one of** `dev` or `prod`. Nothing else in a name is an environment. Add no others such as `test`, `stage` or `local` unless the table below lists them.
4. **Maximum three parts.** If a name needs a fourth part, the area is too broad: split it.
5. **Universal names are fixed.** Use them with exactly these spellings, and don't add synonyms (`serve`, `start:dev`).
6. **Project-specific names** (generators, importers, one-off tools) use an area that describes the project's own thing, such as `generate:entities` or `img:convert`. They never reuse a universal name.
7. **Every project gets** `prepare` (hook setup), `lint`, `typecheck`, `test`, `build`, `dev`.

## Targets

`dev` is the **test server** (drh-mini), and `prod` is production (tertium). In conversation we say "test" for the `dev` target, so the script name is always `dev`.

A task that runs against a server takes the target suffix (`db:backup:dev` runs against the test server). A task that runs only on this machine has no suffix (`dev` runs the app locally).

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
| `deploy` | `:dev`, `:prod` | `deploy:prod` |
| `db` | `:migrate`, `:migrate:dev`, `:backup:dev`, `:backup:prod`, `:sync:dev-to-prod`, `:rotate-password:dev`, `:rotate-password:prod` | `db:backup:dev` |
| `release` | `:patch`, `:minor`, `:major` | `release:minor` |
| `env` | `:sync` (pull the env file from SSM) | `env:sync` |
| `test` | `:integration` | `test:integration` |

Notes on the table:

- `db:migrate` applies the committed migrations to the database the project is configured for. `db:migrate:dev` creates a new migration from schema changes and is for local development only.
- `db:sync:dev-to-prod` overwrites production. It always backs up production first and asks for confirmation.
- `db:rotate-password:*` changes a database user's password and its SSM parameter, and updates the app on the same host.
- `release:*` bumps the version, writes the changelog and publishes.

## Naming a new script

1. Find the area in the table. If it fits, use the next action in that area.
2. If it does not fit, pick a new area, keep it to one word, and add it to this document in a pull request.
3. Put the target last, and only `dev` or `prod`.
4. Check the name against rule 5 before adding it.

## Deviations found in current projects

These names do not follow the convention yet. Rename them in a separate commit per project.

| Project | Current | Convention |
|---|---|---|
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
    "db:backup:dev": "hlpr mariadb backup -ssm-param /env/<project>/env.dev",
    "db:backup:prod": "hlpr mariadb backup -ssm-param /env/<project>/env.prod",
    "db:rotate-password:dev": "bun ./node_modules/@tertium/js/scripts/db-rotate-password/db-rotate-password.ts --ssm-param=/env/<project>/env.dev",
    "deploy:dev": "bun ./node_modules/@tertium/js/scripts/workflow/workflow.ts run deploy -f target=dev",
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
- `:dev` is the test server. Say "test server" when you talk to a person about it.
