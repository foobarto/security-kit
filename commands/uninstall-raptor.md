# /security-kit uninstall-raptor

Remove the raptor framework installed by `/security-kit install-raptor`.

## What to do

### Step 1 — Check lockfile

Read `.security-kit/installed.yaml → raptor.installed`. If false or absent:

```
Raptor is not installed in this project.
```

and stop.

### Step 2 — Confirm

Raptor install is substantial; confirm before removing:

```
Uninstalling raptor will remove:
  - .claude/agents/* (raptor-installed agents)
  - .claude/commands/raptor-*.md and raptor's other commands
  - .claude/skills/<raptor-skill-names>
  - Raptor additions to .claude/settings.json (tagged entries only)
  - The security-kit raptor-managed block in CLAUDE.md
  - .raptor/ directory

Continue? (yes/no)
```

### Step 3 — Remove components

**Agents and commands:** compare against `$SOURCES/raptor/.claude/agents/*` and `.claude/commands/*`. Remove only files that match upstream names.

**Skills:** for every skill in `.claude/skills/` whose symlink target is under `$SOURCES/raptor/`, remove the symlink.

**Settings:** read `.claude/settings.json`; remove every entry tagged with `"_sec_kit_raptor": true`. Remove the tag from the JSON structure. If the file is now empty `{}`, delete it.

**CLAUDE.md:** remove the block between `<!-- BEGIN security-kit raptor-managed section -->` and `<!-- END security-kit raptor-managed section -->` inclusive. If CLAUDE.md is now empty (or only whitespace), delete it.

**.raptor/:** `rm -rf .raptor` — ask the user first if it contains scan output they might want to keep.

### Step 4 — Update lockfile

Set `raptor.installed: false` and remove the `raptor` section's details (keep only the field as a tombstone, or remove the section entirely — either works).

### Step 5 — Report

```
Raptor uninstalled.
  - Agents: removed
  - Commands: removed
  - Skills: removed
  - CLAUDE.md: cleaned (security-kit block removed)
  - .raptor/: removed

Source at $SOURCES/raptor remains for future re-installation.
```

## Safety

- Never delete files that weren't installed by the kit. Compare against `$SOURCES/raptor/` contents to determine what was kit-installed.
- If CLAUDE.md has no `BEGIN security-kit raptor-managed section` marker, refuse to modify it — tell the user raptor was installed some other way and they should clean it up manually.
- If `.raptor/` has files modified after install (check mtime vs lockfile install timestamp), warn before deleting.
