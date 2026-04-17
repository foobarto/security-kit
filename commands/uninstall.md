# /security-kit uninstall [variant]

Remove kit-installed skills from the current project.

## Behavior

- Default (no arg): remove all sec-kit-managed skills (everything in `.security-kit/installed.yaml` → `skills[]`).
- With `variant` arg: only remove skills associated with that variant. Useful for "downgrade full to minimal."

## What to do

### Step 1 — Read lockfile

Read `.security-kit/installed.yaml`. If it doesn't exist, print:

```
No security-kit installation found in this project.
```

and stop.

### Step 2 — Determine targets

- If no variant arg: target = every skill in `installed.yaml → skills[]`.
- If variant arg: read `manifests/<variant>.yaml`, target = that skill list.

### Step 3 — Remove symlinks

For each target skill:

```bash
# Only remove if it's a symlink (never remove user's own dirs)
[ -L ".claude/skills/$name" ] && rm ".claude/skills/$name"
```

Refuse to delete non-symlinks. Print a warning listing any skipped entries.

### Step 4 — Update lockfile

If ALL skills were removed: delete `.security-kit/installed.yaml`. If a subset: update `skills[]` to remove just the targeted entries.

### Step 5 — Clean up empty dirs

- If `.claude/skills/` is empty, remove it.
- If `.claude/` only contains an empty `skills/`, leave it (may hold other Claude config).
- If `.security-kit/` now contains only `prompts/` and no lockfile, prompt: "Also remove .security-kit/ and its prompts? (yes/no)"

### Step 6 — Report

```
Uninstalled 22 skills.
Kept intact:
  - .claude/settings.json (if present; not managed by sec-kit)
  - .claude/hooks/ (use uninstall-hooks separately)
  - raptor (use uninstall-raptor separately)
  - user-owned .claude/skills/* (non-symlink)

Sources at ~/.local/share/security-kit/sources remain. Use `/security-kit purge` to remove those too.
```

## Safety

- Never delete files that aren't symlinks.
- Never delete from `.claude/skills/` any entry not listed in the lockfile.
- Never delete anything under `~/.local/share/security-kit/` unless the user runs `purge` explicitly.

## Purge (optional deep clean)

If the user says "purge" or "clean up completely":

1. Run uninstall, uninstall-hooks, uninstall-raptor on this project.
2. Ask: "Remove sources at `$SOURCES`? This affects every other project using the kit's sources. (yes/no)"
3. If yes: `rm -rf "$SOURCES"`. Confirm with the user first — this is destructive and shared.
