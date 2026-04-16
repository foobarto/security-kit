# /sec-kit uninstall-hooks <name...>

Remove opt-in hooks installed by `/sec-kit install-hooks`.

## What to do

### Step 1 — Parse arguments

Each `<name>` must match an entry in `.security-kit/installed.yaml → hooks[]`. If a requested name isn't installed, skip it with a note.

### Step 2 — Remove from hooks.json

Read `.claude/hooks/hooks.json`. For each target name:

- Find every entry tagged with `"_sec_kit_hook": "<name>"` (added by `install-hooks`)
- Remove just those entries, preserving other entries

If `hooks.json` is empty after removal, delete the file.

### Step 3 — Update lockfile

Remove the hook entries from `installed.yaml → hooks[]`.

### Step 4 — Report

```
Uninstalled hooks: gh-cli, modern-python
Remaining hooks: (none)
```

## Safety

- Never touch hook entries that don't carry the `_sec_kit_hook` tag.
- If `hooks.json` is malformed, refuse to edit and ask the user to fix it by hand.
