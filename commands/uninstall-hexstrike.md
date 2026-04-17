# /security-kit uninstall-hexstrike

Remove the HexStrike AI wrapper skill and deployment files from the current project.

## What to do

### Step 1 — Confirm

```
Removing hexstrike will:

  1. Delete .claude/skills/hexstrike/ (wrapper skill)
  2. Delete .hexstrike/ (deployment files, venv, scripts)
  3. NOT stop the running MCP server — run
     .hexstrike/scripts/hexstrike-stop.sh first if it's running.

Continue? (yes/no)
```

Wait for explicit "yes".

### Step 2 — Remove

```bash
# Stop server if running (best effort)
if [ -f .hexstrike/scripts/hexstrike-stop.sh ]; then
  .hexstrike/scripts/hexstrike-stop.sh 2>/dev/null || true
fi

# Remove wrapper skill
rm -rf .claude/skills/hexstrike

# Remove deployment files (including venv)
rm -rf .hexstrike
```

### Step 3 — Update lockfile

Remove the `hexstrike:` section from `.security-kit/installed.yaml`.

If the file only contains the hexstrike entry, delete it entirely.

### Step 4 — Report

```
HexStrike AI removed from this project.

The source clone remains at $SOURCES/hexstrike-ai for future reinstalls.
To fully remove the source: rm -rf $SOURCES/hexstrike-ai
```
