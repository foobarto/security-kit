# /sec-kit install [variant]

Install a skill bundle into `.claude/skills/` of the current project. No hooks installed.

## Arguments

- `variant` — one of `minimal`, `sec-review`, `threat-modeling`, `full`. Default: `full`.

## What to do

### Step 1 — Resolve the sources directory

Follow the SKILL.md resolution order:
1. `--sources-dir` flag if provided
2. `.security-kit.yaml` in current working directory → `sources_dir:`
3. `~/.config/security-kit/config.yaml` → `sources_dir:`
4. `$SECURITY_KIT_HOME` env var
5. `${XDG_DATA_HOME:-$HOME/.local/share}/security-kit/sources`

Store the resolved path as `SOURCES`.

### Step 2 — Ensure sources exist

For every repo the chosen variant needs, check that its directory exists under `SOURCES`:

- `trailofbits-skills` — from `https://github.com/trailofbits/skills.git`
- `claude-code-owasp` — from `https://github.com/agamm/claude-code-owasp.git`
- `threat-modeling-toolkit` — from `https://github.com/josemlopez/threat-modeling-toolkit.git`

If any are missing:

```bash
mkdir -p "$SOURCES"
cd "$SOURCES"
git clone --depth 1 https://github.com/trailofbits/skills.git trailofbits-skills
git clone --depth 1 https://github.com/agamm/claude-code-owasp.git
git clone --depth 1 https://github.com/josemlopez/threat-modeling-toolkit.git
```

Only clone repos actually required by the chosen variant.

### Step 3 — Read the manifest

Read `manifests/<variant>.yaml`. Each entry has:

```yaml
skills:
  - name: audit-context-building
    repo: trailofbits-skills
    path: plugins/audit-context-building/skills/audit-context-building
```

### Step 4 — Create the project-local layout

```bash
mkdir -p .claude/skills .security-kit/prompts
```

### Step 5 — Symlink each skill

For each skill in the manifest:

```bash
ln -sfn "$SOURCES/$repo/$path" ".claude/skills/$name"
```

Use `-sfn` so re-installing idempotently replaces old symlinks.

### Step 6 — Copy prompt templates

Copy every file from the skill's `prompts/` directory to `.security-kit/prompts/`:

```bash
cp -r "$KIT_PATH/prompts/"* .security-kit/prompts/
```

`KIT_PATH` is the directory holding this skill's `SKILL.md`. Resolve it via `$CLAUDE_PLUGIN_ROOT` if set, or by walking up from this command file.

### Step 7 — Write the lockfile

Write `.security-kit/installed.yaml`:

```yaml
version: 1
variant: <variant>
installed_at: <ISO 8601 timestamp>
sources_dir: <absolute path of SOURCES>
skills:
  - name: audit-context-building
    source: <SOURCES>/trailofbits-skills/plugins/audit-context-building/skills/audit-context-building
  - ...
hooks: []
raptor: false
```

### Step 8 — Summarize for the user

Print:

```
Installed variant 'full': 22 skills
  sources: ~/.local/share/security-kit/sources
  prompts: .security-kit/prompts/
  lockfile: .security-kit/installed.yaml

Hooks: none installed. Use `/sec-kit install-hooks <name>` to opt in.
Raptor: not installed. Use `/sec-kit install-raptor` to opt in.

Run `/sec-kit scan` to execute the installed skills, or `/sec-kit smart-scan` for a tailored prompt.
```

## Idempotence

Re-running `/sec-kit install` with the same variant should be a no-op. Re-running with a different variant should replace symlinks to match the new variant (removing skills not in the new variant, adding skills newly required).

Implementation: before adding symlinks, read the existing lockfile's `skills[]`. For each skill present in the old list but absent from the new manifest, delete its symlink. Then add/update per the new manifest.

## Failure modes

- **Network down on first run** — can't clone sources. Tell the user to either retry online or set `sources_dir:` to an existing location.
- **Existing non-symlink in `.claude/skills/<name>`** — do not overwrite. Report a conflict and ask the user to resolve.
- **`.security-kit/installed.yaml` exists but points at a different variant** — tell the user; wait for explicit confirmation before switching.
