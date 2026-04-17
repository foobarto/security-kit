# /sec-kit uninstall-pentagi

Remove the PentAGI wrapper skill and deployment files from the current project.

## What to do

### Step 1 — Confirm

```
Removing pentagi will:

  1. Delete .claude/skills/pentagi/ (wrapper skill)
  2. Delete .pentagi/ (deployment config, scripts, .env)
  3. NOT stop running Docker containers — you may want to run
     .pentagi/scripts/pentagi-stop.sh first if PentAGI is running.

Continue? (yes/no)
```

Wait for explicit "yes".

### Step 2 — Remove

```bash
# Stop containers if running (best effort)
if [ -f .pentagi/scripts/pentagi-stop.sh ]; then
  .pentagi/scripts/pentagi-stop.sh 2>/dev/null || true
fi

# Remove wrapper skill
rm -rf .claude/skills/pentagi

# Remove deployment files
rm -rf .pentagi
```

### Step 3 — Update lockfile

Remove the `pentagi:` section from `.security-kit/installed.yaml`.

If the file only contains the pentagi entry, delete it entirely.

### Step 4 — Report

```
PentAGI removed from this project.

The source clone remains at $SOURCES/pentagi for future reinstalls.
To fully remove the source: rm -rf $SOURCES/pentagi
```
