# extension-creator

Repository: [github.com/noant/spawn-ext-creator](https://github.com/noant/spawn-ext-creator)

Spawn extension that helps you **author other Spawn extensions** and methodology packs.

Installed extension id: **`extension-creator`** (see `extsrc/config.yaml` → `name`).

## What this extension does

After you install it into a **target project**, you get:

- **Guides** copied under `spawn-ext-guide/` — a full human guide (`user-guide.md`) and compact **machine-oriented** specs in `spawn-ext-guide/ai/` (`core`, `config-yaml`, `skill-sources`, `mcp-json`, `cli`) so agents and humans share the same Spawn packaging rules.
- **Six workflow skills** (see below) your IDE can surface — each pulls in the relevant guide snippets via `required-read`, so the agent follows Spawn’s `extsrc/` layout, `config.yaml`, skills, MCP, and CLI without guessing.
- A practical **pipeline** for new methodologies: design layout → scaffold repo → declare files and modes → write skill sources → optionally wire MCP → validate with `spawn extension check --strict` and a trial install.

Use it whenever you want Cursor (or another Spawn-backed IDE) to help **create or maintain** another Spawn extension or a reusable methodology pack.

## Skills

These skills are defined under `extsrc/skills/` and registered in `config.yaml`. Names are what typically appear in the IDE after Spawn renders them.

| Skill | Purpose |
|-------|---------|
| **`spawn-methodology-shape`** | Shape a methodology before touching YAML: namespaces under `extsrc/files/`, what is **static** vs **artifact**, what goes into global vs skill-local reads. |
| **`spawn-ext-bootstrap`** | Start from zero: run `spawn extension init`, set stable `name` / `version`, plan path prefixes so extensions do not collide in combined installs. |
| **`spawn-ext-config`** | Maintain **`config.yaml`**: every file under `extsrc/files/` declared under `files`, `folders`, read flags, ignores, `setup` hooks, correct `mode`. |
| **`spawn-ext-skill-sources`** | Write **`extsrc/skills/*.md`**, register keys under `skills:`, wire **`required-read`** and naming so merged reads stay consistent. |
| **`spawn-ext-mcp`** | Author **`extsrc/mcp.json`** in Spawn’s `servers[]` format (transport, env placeholders, capabilities, globally unique server names). |
| **`spawn-ext-verify`** | Run **`spawn extension check . --strict`**, smoke-test **`spawn extension add`** in a disposable repo, healthcheck / distribution notes. |

Suggested order for a **new** pack: methodology shape → bootstrap → config → skill sources → (optional) MCP → verify.

## Contents

| Path | Role |
|------|------|
| `extsrc/files/spawn-ext-guide/user-guide.md` | Full human-readable authoring guide |
| `extsrc/files/spawn-ext-guide/ai/*.md` | Compact machine-oriented specs (`core`, `config-yaml`, `skill-sources`, `mcp-json`, `cli`) |
| `extsrc/skills/*.md` | Skill sources shipped with this pack (listed in **Skills**) |

After installation into a target repo, templates are materialized under paths declared in `config.yaml` (e.g. `spawn-ext-guide/...`). Skills are rendered by your IDE adapter according to Spawn rules.

## Prerequisites

- **Spawn CLI** (`spawn`) installed and on `PATH`
- A **target git repository** where you want this extension installed

Install the CLI using the official Spawn distribution instructions for your platform (the exact package or binary name depends on how Spawn is shipped in your environment).

## Installation

Run all commands from the **root of the target repository** (the project that should receive the extension).

1. Initialize Spawn metadata in the target (once per repo):

   ```bash
   spawn init
   ```

2. Install this extension from Git:

   ```bash
   spawn extension add https://github.com/noant/spawn-ext-creator.git
   ```

   Optional branch:

   ```bash
   spawn extension add https://github.com/noant/spawn-ext-creator.git --branch main
   ```

   Or from a **local clone** (same tree):

   ```bash
   spawn extension add /path/to/spawn-ext-creator
   ```

3. Confirm the extension is registered:

   ```bash
   spawn extension list
   ```

You should see **`extension-creator`** (or the id defined by `name` in `extsrc/config.yaml`). Files appear under the paths listed in that manifest (for example `spawn-ext-guide/...`).

## Updating or removing

```bash
spawn extension update extension-creator
spawn extension remove extension-creator
```

Names match `name` in `extsrc/config.yaml`.

## Developing this repo

From this repository root (the extension **source** tree):

```bash
spawn extension check . --strict
```

Fix any reported issues before publishing or sharing the extension.

## Documentation

- Narrative guide: `extsrc/files/spawn-ext-guide/user-guide.md` (also installed into the target when you add this extension).
- CLI checklist and bundle manifests: `extsrc/files/spawn-ext-guide/ai/cli.md`.
