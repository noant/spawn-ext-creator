---
document: spawn-extension-config-yaml
audience: machine
encoding: utf-8
prerequisite: spawn-ext-guide/ai/core.md (vocabulary, static vs artifact, layout constraints)
---

# config.yaml — machine schema

## config_yaml.schema_version

- Top-level `schema`: integer MUST be `1` for current format.

## config_yaml.top_level_keys

| key | required | type | semantics |
|-----|----------|------|-----------|
| schema | yes | int | MUST be 1 |
| version | yes | string | string-compared on update |
| name | strongly recommended | string | id and folder `spawn/.extend/{name}/` |
| files | no | map | target-relative path → file metadata |
| folders | no | map | target-relative dir → folder metadata |
| skills | no | map | skill filename under skills/ → skill metadata |
| agent-ignore | no | list[string] | globs merged into IDE agent-ignore |
| git-ignore | no | list[string] | lines merged into target `.gitignore` |
| setup | no | object | lifecycle phase → script basename under `extsrc/setup/` |

constraint: use ONLY keys above; unknown keys MAY be rejected or warned under strict validation in future.

empty_maps:

- Prefer omitting unused keys entirely; if present, `files`, `folders`, `skills`, `setup` MUST remain valid maps (`{}` allowed where tooling accepts empty maps).

## agent_authoring_invariants

1. **Mirror rule**: every file under `extsrc/files/**` MUST appear as a key under `files:`; key = path relative to **target** root (POSIX). No stray templates; no missing declarations before `--strict`.
2. **Strict descriptions**: if `globalRead` or `localRead` is **`required`** or **`auto`**, `description` MUST be non-empty.
3. **Skills registration**: declare each shipped `extsrc/skills/*.md` under `skills:` whenever you need `required-read`, explicit `name`/`description`, or stable metadata; do not rely on discovery alone for published packs.
4. **Folders vs files**: `folders:` declares directory **policy** (`mode`); templates for real files still require separate `files:` entries (and files on disk under `extsrc/files/`).

## config_yaml.files

key_rules:

- key: path relative to TARGET repo root, POSIX (example: `spec/main.md`).
- template_disk_path: `extsrc/files/<same_relative_path>`.

value_fields:

| field | type | default | semantics |
|-------|------|---------|-----------|
| description | string | optional | under strict: REQUIRED non-empty if `globalRead` or `localRead` is not `no` |
| mode | string | `static` | `static` \| `artifact` |
| globalRead | string | `no` | `required` \| `auto` \| `no` |
| localRead | string | `no` | `required` \| `auto` \| `no` |

mode_semantics:

- `static`: copy template on install; updates MAY overwrite target (typically with warning if existed).
- `artifact`: copy ONLY if target missing; thereafter MUST NOT overwrite on extension update.

globalRead_semantics:

- `required`: listed in `spawn/navigation.yaml` as must-read for session context.
- `auto`: optional contextual in global navigation.
- `no`: omit from global navigation for this entry.

localRead_semantics:

- `required`: mandatory reads in rendered skills for THIS extension.
- `auto`: contextual reads block in rendered skills.
- `no`: omit from local skill read lists from this file entry.

required_read_merge:

- `skills[].required-read`: mandatory paths for THAT skill only.
- Spawn merges global required + local required + skill `required-read`, deduplicates.
- Duplication vs `globalRead: required` / `localRead: required` redundant but allowed.

## config_yaml.folders

- key: directory path relative to target root.
- value: object with `mode`: `static` \| `artifact` (implementations MAY default `static` if omitted).
- purpose: declare ownership for removal/update policy; empty-dir creation MAY depend on tooling.

## config_yaml.skills

- key: exact filename in `extsrc/skills/` including `.md` (example: `run-task.md`).

value_fields:

| field | type | semantics |
|-------|------|-----------|
| name | string optional | display id; overrides frontmatter and filename heuristic |
| description | string optional | overrides frontmatter |
| required-read | list[string] | target-relative paths mandatory for this skill (additive) |

skill body format: see `spawn-ext-guide/ai/skill-sources.md`.

## config_yaml.agent_ignore_and_git_ignore

- `agent-ignore`: glob strings → IDE ignore for agents.
- `git-ignore`: lines appended to target `.gitignore`.

## config_yaml.setup

- each value: filename under `extsrc/setup/`; if key present, file MUST exist when strict check runs.

phases:

| yaml_key | phase |
|----------|--------|
| before-install | before copying extension into target |
| after-install | after install materialization |
| before-uninstall | before removal; failures MAY block when hook defined |
| after-uninstall | after removal |
| healthcheck | `spawn extension healthcheck <name>` from target |

scripts: trusted code; SHOULD be idempotent; MUST NOT silently delete artifact user data; MUST fail loudly on unsafe migration.

## annotated_config_yaml_example

```yaml
schema: 1
name: acme-methodology
version: "1.0.0"
files:
  methodology/guide.md:
    description: "Core methodology; read when using Acme skills."
    mode: static
    globalRead: auto
    localRead: required
  spec/design/notes.md:
    description: "Team architecture notes (per-repo)."
    mode: artifact
    globalRead: no
    localRead: auto
folders:
  spec/tasks:
    mode: artifact
skills:
  acme-run-task.md:
    name: acme-run-task
    description: "Execute the Acme task flow in the current repo."
    required-read:
      - methodology/guide.md
agent-ignore:
  - .acme/cache/**
  - spec/tasks/**/.drafts/**
  - methodology/.local/**
git-ignore:
  - .acme/cache/**
  - spec/tasks/**/.drafts/**
setup:
  after-install: after_install.py
  healthcheck: healthcheck.py
```

matching_extsrc_paths:

- `extsrc/files/methodology/guide.md`
- `extsrc/files/spec/design/notes.md`
- `extsrc/skills/acme-run-task.md`
- optional `extsrc/setup/after_install.py`, `extsrc/setup/healthcheck.py`
- optional `extsrc/mcp.json` (see `spawn-ext-guide/ai/mcp-json.md`)
