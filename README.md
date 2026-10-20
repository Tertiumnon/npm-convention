# NPM commands convention

One npm script name for the same task in every project.

Names for `package.json` scripts, so the same task has the same name in every project. `deploy:prod` means the same everywhere.

Repository: https://github.com/Tertiumnon/npm-convention

## Required fields

- `package.json`: `name`, `version`, `description` (one sentence), `author` (an object with `name`, `email` and `url`), `license`, `repository` (the GitHub URL), and the release and `version` scripts. Also set `homepage` and `keywords` so the package is findable on npmjs.

## Environments

| Where | Environment | Used by |
|---|---|---|
| (none) | Local: the developer's own machine | developer |
| `dev` | Developer's remote environment | developer |
| `test` | QA engineer's remote environment | QA, automated tests |
| `preprod` | Team's remote environment; last check before release | team |
| `prod` | Production | users |

Changes move in this order: `dev` → `test` → `preprod` → `prod`. A project may skip an environment it lacks.

A script reaches only the environment in its where part: `dev` scripts reach `dev`, not the others. A task with no where part runs on the developer's machine.

## Scripts

### Name format

`action[:what][:where]`

1. Lowercase, colon-separated. Hyphens inside a word (`rotate-password:db:prod`).
2. Action first, what second, where last: `backup:db:prod`, not `db:backup:prod`.
3. Where is one of `dev`, `test`, `preprod`, `prod`. Nothing else is an environment.
4. Maximum three parts. A fourth part means the action is too broad: split it.
5. Universal names keep their exact spelling. No synonyms (`serve`, `start:dev`).
6. Project-specific names use an action and a what that describe the project's own thing (`generate:entities`, `convert:img`). They never reuse a universal name.
7. Every project has `prepare`, `lint`, `typecheck`, `test`, `build`, `dev`.

Parts are positional, and the where part is always one of the fixed environments. So `deploy:prod` has no what, and `release:minor` has no where.

### Universal names

Single-purpose, exact spelling. They don't follow rule 2.

| Script | Meaning |
|---|---|
| `dev` | Run locally with watch and reload |
| `start` | Run the built or production entry point |
| `build` | Produce build output |
| `lint` | All static checks: runs each `lint:<language>` in sequence |
| `fix` | All automatic fixes: runs each `fix:<language>` in sequence |
| `typecheck` | Type checks without emitting files |
| `test` | Unit tests (fast, no network) |
| `test:integration` | Tests that need running services |
| `prepare` | Install git hooks: `git config core.hooksPath .githooks`. No error suppression, so a missing `.git` fails the install |

### Per-language checks

The what part is the file type: `css`, `ts`, `js`, `json`, `md`.

- `lint:<language>` runs the linter and formatter in check mode. Formatting is part of linting, so there is no `format` script.
- `fix:<language>` applies that language's fixes.
- `lint` and `fix` chain the per-language scripts with `&&`. Adding a language adds one term to each chain.

### Action names

| Action            | What                      | Where                            | Example                        |
|-------------------|---------------------------|----------------------------------|--------------------------------|
| `deploy`          |                           | `dev`, `test`, `preprod`, `prod` | `deploy:preprod`               |
| `migrate`         | `db`                      | environments; none = local       | `migrate:db`, `migrate:db:dev` |
| `generate`        | `migration`               | none (local only)                | `generate:migration`           |
| `backup`          | `db`, `img`               | environments                     | `backup:db:prod`               |
| `rotate-password` | `db`                      | environments                     | `rotate-password:db:prod`      |
| `sync`            | `img`, `env`              | environments for `img`           | `sync:env`                     |
| `release`         | `patch`, `minor`, `major` |                                  | `release:minor`                |
| `lint`            | `<language>`              |                                  | `lint:css`                     |
| `fix`             | `<language>`              |                                  | `fix:css`                      |

Notes:

- `migrate:db` applies committed migrations to the database of its environment.
- `generate:migration` creates a migration from schema changes. Developer's machine only.
- `rotate-password:db:<environment>` changes a database user's password and its stored configuration, and updates the application on the same host.
- `sync:env` pulls the environment's configuration file.
- `release:*` bumps the version, writes the changelog and publishes.

### Example

Names only. Values are left out on purpose.

```json
{
  "scripts": {
    "dev": "...",
    "start": "...",
    "build": "...",
    "lint": "lint:css && lint:ts",
    "lint:css": "...",
    "lint:ts": "...",
    "fix": "fix:css && fix:ts",
    "fix:css": "...",
    "fix:ts": "...",
    "typecheck": "...",
    "test": "...",
    "test:integration": "...",
    "prepare": "git config core.hooksPath .githooks",
    "migrate:db": "...",
    "generate:migration": "...",
    "backup:db:prod": "...",
    "deploy:test": "...",
    "deploy:preprod": "...",
    "deploy:prod": "...",
    "release:patch": "...",
    "release:minor": "...",
    "release:major": "..."
  }
}
```

### Naming a new script

1. Find the action in the table. If it fits, add the what and where it needs, in that order.
2. If not, pick a new one-word action, and add it to this document in a pull request.
3. Put the where last, using one of the four environments.
4. Check the name against rules 1 to 6.

## Open decisions

- **QA's remote environment.** It is `test`. Projects that call it `dev` need renaming: `deploy:dev` → `deploy:test`, `<action>:db:dev` → `<action>:db:test`, and the deploy workflow's target choices. Until then, read `dev` in such a project as "the QA environment".
- **`preprod`.** Reserved. Add it when a team environment exists.

## Deviations

Rename in a separate commit per project.

| Current                                                                             | Convention               |
|-------------------------------------------------------------------------------------|--------------------------|
| `deploy:dev`, `<action>:db:dev` in a project whose remote environment is used by QA | `test` in the where part |

## For AI assistants

- Use the names here. Don't invent a name when a table entry fits.
- Universal names exactly. Never `serve` or `start:dev`.
- Order is action, what, where: `backup:db:prod`.
- Lint: add `lint:<language>` and `fix:<language>`, chain them in `lint` and `fix`. Never add `format`.
- Where is `dev`, `test`, `preprod` or `prod`, in that promotion order. No where means the developer's machine.
