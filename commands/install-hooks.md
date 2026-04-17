# /security-kit install-hooks <name...>

Opt-in installation of Trail of Bits skills that ship with hooks. Whitelist-only.

## Whitelist

Only these hooks can be installed via this command — anything else is refused:

| Name | What it does | Risk |
|---|---|---|
| `gh-cli` | SessionStart: prepend PATH shim; PreToolUse: intercept WebFetch/Bash on github.com URLs and redirect to `gh` CLI | Low — purely a convenience, no network, no cred reads |
| `modern-python` | SessionStart: prepend PATH shim that suggests `uv` when `pip` is invoked | Low — advisory only |

Deliberately not on the whitelist:

- `skill-improver` — aggressive self-continuing Stop-hook loop (burns tokens, noisy)
- `firebase-apk-scanner` — active internet scanner
- `second-opinion` — auto-launches `codex mcp-server` if present (third-party MCP)

Any of these can still be installed manually by the user; `install-hooks` just refuses to automate it.

## What to do

### Step 1 — Validate arguments

For each `<name>` argument, confirm it is in the whitelist. If not, refuse with a message explaining why (pointer to this file for the whitelist).

### Step 2 — Ensure `trailofbits-skills` is cloned

If `$SOURCES/trailofbits-skills/` is missing, clone it (same as `install`).

### Step 3 — Warn and confirm

For each hook being installed, print exactly what it adds:

```
Installing hook: gh-cli
  Adds to .claude/hooks/hooks.json:
    - SessionStart → prepends trailofbits-skills/plugins/gh-cli/hooks/shims to PATH
    - PreToolUse (WebFetch, Bash) → redirects github.com URLs to gh CLI
  Modifies: PATH for Claude's shell tool calls in this project
  Reversible: yes, via /security-kit uninstall-hooks gh-cli
```

Wait for explicit "yes" from the user before proceeding (Claude should ask).

### Step 4 — Install

For each confirmed hook, merge its `hooks.json` into `.claude/hooks/hooks.json` of the project:

- Read `$SOURCES/trailofbits-skills/plugins/<name>/hooks/hooks.json`
- Read existing `.claude/hooks/hooks.json` (or start with `{}` if absent)
- Merge the hook entries under their event keys (`SessionStart`, `PreToolUse`, etc.)
- Tag each merged entry with `"_sec_kit_hook": "<name>"` so uninstall can find and remove only its own additions

### Step 5 — Update lockfile

Append to `.security-kit/installed.yaml`:

```yaml
hooks:
  - name: gh-cli
    installed_at: <timestamp>
```

### Step 6 — Report

```
Installed hooks: gh-cli
Claude Code will pick up the new hook configuration on next session start in this directory.
To remove: /security-kit uninstall-hooks gh-cli
```
