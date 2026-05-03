# Extension Creator (Spawn extension)

Extension Creator helps you **design and maintain Spawn extensions** — reusable methodology packs for Spawn-backed editors. It ships guides and eight companion skills so people and assistants follow one packaging workflow. Add it to your Git repository with the Spawn CLI.

## Install

### 1. Install Spawn CLI (uv tool)

Install [uv](https://docs.astral.sh/uv/), then install the Spawn command from PyPI into uv’s tools directory (**`spawn` must be on `PATH`** after install — see uv’s docs for tool paths):

```bash
uv tool install spawn-cli
```

Upstream source repository: **[github.com/noant/spawn-cli](https://github.com/noant/spawn-cli)**.

### 2. Initialize the repo and add Extension Creator

From the **root of the target git repository**, initialize Spawn once per repo, then install this extension:

```bash
spawn init
spawn extension add https://github.com/noant/spawn-ext-creator.git
```

One line (same result without a prior global install; run from that repo root):

```bash
uvx --from spawn-cli spawn init && uvx --from spawn-cli spawn extension add https://github.com/noant/spawn-ext-creator.git
```

### Update or remove

```bash
spawn extension update extension-creator
spawn extension remove extension-creator
```

Extension ids match **`name`** in this pack’s `extsrc/config.yaml`.

## How it works

You use Extension Creator when Cursor (or another Spawn-backed IDE) should **create or maintain** another Spawn extension or reusable methodology pack without guessing `extsrc/` layout, `config.yaml`, skills, MCP, or CLI checks.

The practical pipeline:

1. **Shape** the methodology — namespaces under `extsrc/files/`, what is **static** vs **artifact**, what belongs in global reads vs skill-local reads (**spawn-methodology-shape**).
2. **Bootstrap** the repo — `spawn extension init`, stable **`name`** / **`version`**, path prefixes that avoid collisions when multiple extensions install together (**spawn-ext-bootstrap**).
3. **Declare** everything in **`config.yaml`** — every file under `extsrc/files/`, folders, read flags, ignores, **`setup`**, correct **`mode`** (**spawn-ext-config**).
4. **Author skills** — `extsrc/skills/*.md`, register keys under **`skills:`**, wire **`required-read`** so merges stay consistent (**spawn-ext-skill-sources**).
5. Optionally **wire MCP** — **`extsrc/mcp/windows.json`**, **`linux.json`**, **`macos.json`** in Spawn’s **`servers[]`** shape per file (**spawn-ext-mcp**).
6. **Verify** — `spawn extension check . --strict` and a disposable **`spawn extension add`** smoke test (**spawn-ext-verify**).
7. Before publishing an update, **bump `version`** and re-verify (**spawn-ext-increment-version**).

Suggested order for a **new** pack: methodology shape → bootstrap → config → (optional) hints → skill sources → (optional) MCP → verify. Before a release: increment version → verify.

## The `spawn-ext-guide/` layout

After install, Spawn materializes the paths declared in `extsrc/config.yaml`. Human narrative and compact machine specs live side by side so agents and authors share one rule set.

- **`spawn-ext-guide/user-guide.md`** — full standalone authoring guide (narrative across all topics).
- **`spawn-ext-guide/ai/core.md`** — machine baseline: `extsrc` tree, static vs artifact, naming and uniqueness, install outputs (**`localRead: required`** — baseline for merged reads).
- **`spawn-ext-guide/ai/config-yaml.md`** — `config.yaml` schema: files, folders, skills modes, reads, ignores, setup.
- **`spawn-ext-guide/ai/skill-sources.md`** — skill source markdown, frontmatter, **`required-read`**, rendered skill shape.
- **`spawn-ext-guide/ai/mcp-json.md`** — per-platform **`extsrc/mcp/*.json`** (**servers**, transport, env placeholders, aligned **`name`** sets).
- **`spawn-ext-guide/ai/cli.md`** — CLI commands, bundle shapes, authoring checklist.

Skill sources that ship **with** this pack live under **`extsrc/skills/*.md`** in the Git repo and are registered under **`skills:`** in `extsrc/config.yaml`; your IDE renders them according to Spawn rules.

## Core principles

**Config is the contract.** Every path under `extsrc/files/` must appear under **`files`** (or **`folders`**) with the right **`mode`** and read flags — agents trained on these guides assume the manifest matches the tree.

**Static vs artifact.** This extension ships guides as **`static`** templates. Methodology packs you author may use **`artifact`** for project-owned paths (user data, tasks, seeds) so updates do not clobber working files — declare that explicitly when shaping the pack.

**Skills carry context.** Each skill lists **`required-read`** snippets under `spawn-ext-guide/` so the agent loads the same rules you would open manually; avoid duplicating long prose inside skill bodies when a guide section already exists.

**Verify before publish.** **`spawn extension check . --strict`** plus a trial **`spawn extension add`** in a throwaway repo catches path drift and manifest mistakes early.

**Stable identity, semver versions.** Keep **`name`** fixed across releases; bump **`version`** when you ship observable changes (**spawn-ext-increment-version**).

## Skills

After install, invoke these **skills** by name; Spawn renders them into your IDE’s skill/rules layout.

| Skill | Purpose |
|--------|--------|
| **spawn-methodology-shape** | Shape a methodology before YAML: namespaces under `extsrc/files/`, **static** vs **artifact**, global vs skill-local reads. |
| **spawn-ext-bootstrap** | Start from zero: `spawn extension init`, stable **`name`** / **`version`**, collision-safe path prefixes. |
| **spawn-ext-config** | Maintain **`config.yaml`**: declare every file under `extsrc/files/`, **`folders`**, read flags, ignores, **`setup`**, **`mode`**. |
| **spawn-ext-hints** | Declare optional **`hints.global`** / **`hints.local`** lists (plain strings) merged into navigation, rendered skills, and **`AGENTS.md`** where applicable. |
| **spawn-ext-skill-sources** | Write **`extsrc/skills/*.md`**, register under **`skills:`**, wire **`required-read`** for consistent merges. |
| **spawn-ext-mcp** | Author **`extsrc/mcp/*.json`** — three OS files; **`servers[]`** transport, env placeholders, capabilities; globally unique server **`name`**s with identical name sets across platforms. |
| **spawn-ext-verify** | Run **`spawn extension check . --strict`**, smoke-test **`spawn extension add`** in a disposable repo. |
| **spawn-ext-increment-version** | Bump **`version`** in **`extsrc/config.yaml`** for releases (semver-oriented); keep **`name`** stable. |

## Navigation and reads

**`spawn/navigation.yaml`** (generated by Spawn after install) is the merged index of what agents should read first (`read-required` / contextual reads). Guide fragments under **`spawn-ext-guide/ai/`** use **`localRead: required`** or **`auto`** in `extsrc/config.yaml` so they participate in that merge without hand-maintaining a separate navigation file.

When you author **another** extension, declare its readable paths in **that** pack’s `extsrc/config.yaml` with the appropriate **`globalRead`** / **`localRead`** — same idea as registering extra design docs or methodology paths so combined installs stay coherent.

---

From **this** repository root (extension **source** tree), validate before publishing:

```bash
spawn extension check . --strict
```

## License

MIT — see [LICENSE](LICENSE).
