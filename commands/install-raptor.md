# /security-kit install-raptor

Opt-in installation of the raptor offensive/defensive security framework (by Gadi Evron et al).

## Why this is a separate command

Every other skill in the kit is pure-markdown drop-in. Raptor is different:

- Owns a custom `CLAUDE.md` (~400 lines) that becomes the project's system-prompt context
- Registers `/raptor`, `/raptor-scan`, `/raptor-fuzz`, `/raptor-web`, `/scan`, `/fuzz`, `/web`, `/agentic`, `/oss-forensics` slash commands
- Ships a `SessionStart` hook that writes `.claude/raptor.env` and prints a banner
- Real leverage comes from raptor's Python CLI (`raptor.py agentic --repo <path>`) that orchestrates Semgrep + CodeQL + Claude

These changes are invasive. They should not be automatic — hence this separate command.

## Prerequisites

- Python 3.8+
- `pip install --user -r requirements.txt` from raptor's repo (requests, pydantic, instructor, tabulate)
- Optional but valuable: `semgrep` (`pip install --user semgrep`), `codeql` (download from GitHub)

## What to do

### Step 1 — Ensure raptor source is cloned

If `$SOURCES/raptor/` is missing:

```bash
git clone --depth 1 https://github.com/gadievron/raptor.git "$SOURCES/raptor"
```

**Do not initialize the `.claude/skills/SecOpsAgentKit` submodule** — it's a separate org (`AgentSecOps`) with its own trust boundary. Leave it un-initialized.

### Step 2 — Warn the user explicitly

Print the full impact of raptor installation:

```
Installing raptor will:

  1. Merge raptor's .claude/ into this project's .claude/:
     - .claude/agents/* (17 agents)
     - .claude/commands/* (20 slash commands, incl. /raptor-scan, /fuzz, /web)
     - .claude/settings.json additions (tool-scoped Bash permissions for libexec/raptor-*)
     - A SessionStart hook that sources libexec/raptor-session-init and prints a banner

  2. Create or update project-level CLAUDE.md with raptor's ~400 lines of offensive-security
     directives. If you have an existing CLAUDE.md, raptor's content will be APPENDED (not
     replaced), with a clear begin/end marker for clean removal.

  3. Install a .raptor/ directory in this project for raptor's own state (session dirs,
     scan output, etc.)

Fully reversible via /security-kit uninstall-raptor.

Continue? (yes/no)
```

Wait for explicit "yes".

### Step 3 — Install

```bash
# Merge .claude/ contents
mkdir -p .claude/agents .claude/commands
cp -r "$SOURCES/raptor/.claude/agents/"* .claude/agents/
cp -r "$SOURCES/raptor/.claude/commands/"* .claude/commands/

# Symlink raptor skills
mkdir -p .claude/skills
for skill_dir in "$SOURCES/raptor/.claude/skills/"*/; do
  skill_name=$(basename "$skill_dir")
  [ "$skill_name" = "SecOpsAgentKit" ] && continue  # excluded — separate org
  ln -sfn "$skill_dir" ".claude/skills/$skill_name"
done

# Merge settings.json (additive, tagged entries)
# — read existing .claude/settings.json
# — add raptor's hook entries under SessionStart
# — add raptor's tool-scoped permissions
# — tag every added entry with "_sec_kit_raptor": true for clean uninstall

# Append to CLAUDE.md with markers
cat >> CLAUDE.md <<'MARK_BEGIN'

<!-- BEGIN security-kit raptor-managed section — do not edit between these markers -->
MARK_BEGIN
cat "$SOURCES/raptor/CLAUDE.md" >> CLAUDE.md
cat >> CLAUDE.md <<'MARK_END'
<!-- END security-kit raptor-managed section -->
MARK_END

# Create raptor working dir
mkdir -p .raptor
```

### Step 4 — Update lockfile

```yaml
raptor:
  installed: true
  installed_at: <timestamp>
  source: <SOURCES>/raptor
```

### Step 5 — Report

```
Raptor installed.

Quick commands (try now or after restarting Claude Code in this directory):
  /raptor-scan <path>   # semgrep + codeql + manual verification, produces PoCs + patches
  /raptor-fuzz <binary> # afl++ fuzzing (requires afl++ installed)
  /raptor-web <url>     # web app testing
  /agentic              # autonomous security research mode
  /oss-forensics        # supply-chain forensics (requires BigQuery for full value)

Raptor's SessionStart hook will print a banner on your next session start in this directory.

To remove everything: /security-kit uninstall-raptor
```

### Step 6 — Alert on missing dependencies

Check for `semgrep` and `codeql` in PATH. If missing, print:

```
Note: semgrep not found. Install with:   pip install --user semgrep
Note: codeql not found. Download from:   https://github.com/github/codeql-cli-binaries/releases
Raptor will still work with Claude-only analysis, but the static-analysis layer won't contribute.
```

## Submodule note

If the user later wants raptor's `SecOpsAgentKit` submodule, they can init it manually with:

```bash
cd "$SOURCES/raptor"
git submodule update --init .claude/skills/SecOpsAgentKit
```

Document this in the install-raptor output as "advanced — audit the submodule before enabling."
